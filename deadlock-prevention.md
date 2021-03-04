## PostgreSQL DeadLock Prevention

A deadlock situation on a resource can arise only if all of the following conditions hold simultaneously (Coffman conditions):  

- Mutual exclusion
- Hold and wait or resource holding
- No preemption
- Circular wait

Most deadLock prevention approaches work by preventing one of the four Coffman conditions from occurring. 
In this case study we will play with two of them - **the hold and wait** and **circular wait**.  

Prevention of **hold and wait** can be gained by requiring processes to request all the resources they will need before starting up.
To avoid **circular waits**, resources may be ordered and we can ensure that each process can request resources only in an increasing order of these numbers.

Let's make a table to concurrently update the data from the two transactions:

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

One of the simplest approaches to prevent DeadLock is to divide one transaction into many so that no transaction would wait for another to complete.

<table>
  <thead>
    <th>Client1</th>
    <th>Client2</th>
  </thead>
  <tbody>
    <tr>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION;
        </pre>
      </td>
      <td></td>
    </tr>
    <tr>
      <td></td>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION;
        </pre>
      </td>
    </tr>
    <tr>
      <td>
        <pre>
psql> UPDATE roles SET role='Frodo Baggins'  WHERE name='Ian McKellen';
UPDATE 1
<br>
psql> COMMIT;
COMMIT
        </pre>
      </td>
      <td></td>
    </tr>
    <tr>
      <td></td>
      <td>
        <pre>
psql> UPDATE roles SET role='Boromir'  WHERE name='Elijah Wood';
UPDATE 1
<br>
psql> COMMIT;
COMMIT
        </pre>
      </td>
    </tr>
    <tr>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION 
<br>   
psql> UPDATE roles SET role='Gandalf'  WHERE name='Elijah Wood';
UPDATE 1
<br>
psql> COMMIT;
COMMIT
        </pre>
      </td>
      <td></td>
    </tr>
    <tr>
      <td></td>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION 
<br>
psql> UPDATE roles SET role='Gollum'  WHERE name='Ian McKellen';
UPDATE 1
<br>
psql> COMMIT;
COMMIT
        </pre>
      </td>
    </tr>
    <tr>
      <td></td>
      <td></td>
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

Use SELECT FOR UPDATE to request all the resources the transactions will need before starting up.

<table>
  <thead>
    <th>Client1</th>
    <th>Client2</th>
  </thead>
  <tbody>
    <tr>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION;
        </pre>
      </td>
      <td></td>
    </tr>
    <tr>
      <td></td>
      <td>
        <pre>
psql> START TRANSACTION;
START TRANSACTION;
        </pre>
      </td>
    </tr>
    <tr>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
   
