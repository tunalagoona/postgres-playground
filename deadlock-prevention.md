## Deadlock Prevention

[Reference: deadlock (Wikipedia)](https://en.wikipedia.org/wiki/Deadlock)  
[Reference: lock hierarchies to avoid deadlocks](https://ncona.com/2019/01/lock-hierarchies-to-avoid-deadlocks/)

For the deadlock to happen, all of the following conditions must hold:
* Mutual exclusion
* Hold and wait
* No preemption
* Circular wait

To prevent the deadlock, it is enough to avert one of these conditions; here we consider averting **hold and wait** and **circular wait**.  

To avert the **hold and wait** condition, we can require the processes to request all the resources they need before starting up or by replacing multiple fine-grained locks with a single coarse-grained lock.

To avert the **circular wait** condition, we can order the resources and make each process to request resources only in the increasing order, essentially mimicking the lock hierarchy.

### Database structure:

```
psql> CREATE TABLE roles (  
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  
  name VARCHAR,  
  role VARCHAR  
);

psql> INSERT INTO roles (name, role) VALUES ('Ian McKellen', 'Gandalf');   
psql> INSERT INTO roles (name, role) VALUES ('Elijah Wood', 'Frodo Baggins');
psql> INSERT INTO roles (name, role) VALUES ('Sean Bean', 'Boromir');
psql> INSERT INTO roles (name, role) VALUES ('Andy Serkis', 'Gollum');
```
  
| id      | name | role |
| ----------- | ----------- | ----------- |
|1|Ian McKellen|Gandalf|
|2|Elijah Wood|Frodo Baggins|  
|3|Sean Bean|Boromir| 
|4|Andy Serkis|Gollum| 
   
### Multiple transactions

One of the simplest approaches to prevent the deadlock from happenning is to divide a single transaction into many so that no transaction would wait for another to complete. This approach prevents the **hold and wait** condition.

<table>
  <thead>
    <th>#</th>
    <th>Client#1</th>
    <th>Client#2</th>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION;
        </pre>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>2</td>
      <td></td>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION;
        </pre>
      </td>
    </tr>
    <tr>
      <td>3</td>
      <td>
        <pre>
psql> UPDATE roles SET role='Frodo Baggins'    
      WHERE name='Ian McKellen';
UPDATE 1
<br>
psql> COMMIT;
COMMIT
        </pre>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>4</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE roles SET role='Boromir'     
      WHERE name='Elijah Wood';
UPDATE 1
<br>
psql> COMMIT;
COMMIT
        </pre>
      </td>
    </tr>
    <tr>
      <td>5</td>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION 
<br>   
psql> UPDATE roles SET role='Gandalf'     
      WHERE name='Elijah Wood';
UPDATE 1
<br>
psql> COMMIT;
COMMIT
        </pre>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>6</td>
      <td></td>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION 
<br>
psql> UPDATE roles SET role='Gollum'     
      WHERE name='Ian McKellen';
UPDATE 1
<br>
psql> COMMIT;
COMMIT
        </pre>
      </td>
    </tr>
  </tbody>
</table>
    

  
| id      | name | role |
| ----------- | ----------- | ----------- |
|1|Ian McKellen|Gollum|
|2|Elijah Wood|Gandalf|  
|3|Sean Bean|Boromir| 
|4|Andy Serkis|Gollum| 


### Explicit global lock

We can make an auxiliary table carrying coarse-grained locks, which the clients will need to acquire before issuing any changes. This approach also prevents the **hold and wait** condition.

```
psql> CREATE TABLE locks (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, 
  name VARCHAR
);

psql> INSERT INTO locks (name) VALUES ('global lock');
```

  
| id      | name | 
| ----------- | ----------- | 
|1|global lock|

<table>
  <thead>
    <th>#</th>
    <th>Client#1</th>
    <th>Client#2</th>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION;
        </pre>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>2</td>
      <td></td>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION;
        </pre>
      </td>
    </tr>
    <tr>
      <td>3</td>
      <td>
        <pre>
psql> SELECT * FROM locks    
      WHERE name='global lock' FOR UPDATE;
<br>
 id |    name
----+-------------
  1 | global lock
        </pre>
        <i>The <b>client#1</b> acquires the "global lock" <br />
          to be able to edit the rows in the roles table.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>4</td>
      <td></td>
      <td>
        <pre>
psql> SELECT * FROM locks    
      WHERE name='global lock' FOR UPDATE;
...
        </pre>
        <i>The <b>client#2</b> falls into the waiting mode so it can not continue <br />
          until the global lock is released by the <b>client#1</b>.</i>
      </td>
    </tr>
    <tr>
      <td>5</td>
      <td>
        <pre>
psql> UPDATE roles SET role='Boromir'    
      WHERE name='Andy Serkis';
UPDATE 1
<br>
psql> UPDATE roles SET role='Frodo Baggins'   
      WHERE name='Sean Bean';
UPDATE 1
<br>
psql> COMMIT;
COMMIT
        </pre>
        <i>The <b>client#1</b> releases the global lock.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>6</td>
      <td></td>
      <td>
        <pre>
...
 id |    name
----+-------------
  1 | global lock
        </pre>
        <i>The <b>client#2</b> transaction acquires the lock <br />
          and gets the right to edit the rows in the table.</i>
        <pre>
psql> UPDATE roles SET role='Frodo Baggins'     
      WHERE name='Andy Serkis';
UPDATE 1
<br>
psql> UPDATE roles SET role='Gollum'    
      WHERE name='Sean Bean';
UPDATE 1
<br>
psql> COMMIT;
COMMIT
        </pre>
      </td>
    </tr>
  </tbody>
</table>
   

| id      | name | role |
| ----------- | ----------- | ----------- |
|1|Ian McKellen|Gollum|
|2|Elijah Wood|Gandalf|  
|3|Sean Bean|Gollum| 
|4|Andy Serkis|Frodo Baggins| 



### Lock hierarchy

A lock hierarchy consists of designing an application in a way that locks can be only acquired in a specific order.
This approach prevents **circular waits** condition.

Let's make the clients to update the rows in a sorted order. <b>client#1</b> transaction is going to update rows with id=1 and id=2, and <b>client#2</b> transasction - rows with id=2 and id=3.

<table>
  <thead>
    <th>#</th>
    <th>Client#1</th>
    <th>Client#2</th>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION;
        </pre>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>2</td>
      <td></td>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION;
        </pre>
      </td>
    </tr>
    <tr>
      <td>3</td>
      <td>
        <pre>
psql> UPDATE roles SET role='Tauriel'  WHERE id=1;
UPDATE 1
        </pre>
        <i><b>client#1</b> transaction acquires lock <br />
          on the row with id=1.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>4</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE roles SET role='Galadriel'  WHERE id=2;
UPDATE 1
        </pre>
        <i><b>client#2</b> transaction acquires lock <br />
          on the row with id=2.</i>
      </td>
    </tr>
    <tr>
      <td>5</td>
      <td>
        <pre>
psql> UPDATE roles SET role='Sam'  WHERE id=2;
...
        </pre>
        <i><b>client#1</b> transaction tries to acquire the lock <br />
          on the row with id=2 and falls in a waiting mode.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>6</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE roles SET role='Ork 1'  WHERE id=3;
UPDATE 1
<br />
psql> COMMIT;
COMMIT
        </pre>
        <i><b>client#2</b> transaction acquires the lock <br />
          on the row with id=3, completes the query and commits, <br />
          releasing the lock on the row with id=2.</i>
      </td>
    </tr>
    <tr>
      <td>7</td>
      <td>
        <pre>
UPDATE 1
<br />
psql> COMMIT;
COMMIT
        </pre>
        <i><b>client#1</b> transaction acquires the lock <br />
          on the row with id=2, completes the query and commits.</i>
      </td>
      <td></td>
    </tr>
  </tbody>
</table>


