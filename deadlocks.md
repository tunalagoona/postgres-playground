
## Deadlocks

[Reference](https://www.postgresql.org/docs/9.1/explicit-locking.html)

### Database Structure

```
psql> CREATE TABLE kids (  
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  
  name VARCHAR,  
  age INTEGER  
);

psql> INSERT INTO kids (name, age) VALUES ('Ann', 7);   
psql> INSERT INTO kids (name, age) VALUES ('Ben', 12);
psql> INSERT INTO kids (name, age) VALUES ('Sam', 5);
```

  
| id      | name | age |
| ----------- | ----------- | ----------- |
|1|Ann|7|
|2|Ben|12|  
|3|Sam|5| 
   

### Deadlock on implicit exclusive row-level locks (ordinary UPDATE)
   
   
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
START TRANSACTION
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
START TRANSACTION
        </pre>
      </td>
    </tr>
    <tr>
      <td>3</td>
      <td>
        <pre>
psql> UPDATE kids SET age=10 WHERE name='Ann';
UPDATE 1
        </pre>
        <i>
          An exclusive row-level lock is acquired by the <b>client#1</b>.<br/>
          The lock is held until the transaction is completed.
        </i>
      </td>
      <td> </td>
    </tr>
    <tr>
      <td>4</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE kids SET age=13 WHERE name='Ben';
UPDATE 1
        </pre>
        <i>
        An exclusive row-level lock is acquired by the <b>client#2</b>.<br />
        The lock is held until the transaction is completed.
        </i>
      </td>
    </tr>
    <tr>
      <td>5</td>
      <td>
        <pre>
psql> UPDATE kids SET age=9 WHERE name='Ben';
...
        </pre>
        <i>The query will wait until the <b>client#2</b> releases the lock.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>6</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE kids SET age=5 WHERE name='Ann';

ERROR:  deadlock detected
DETAIL:  Process 37184 waits for ShareLock <br/>
on transaction 17500; blocked by process 37281.<br/>
Process 37281 waits for ShareLock on transaction 17501; <br/>
blocked by process 37184.
        </pre>
        <i>Circular wait situation: PostgreSQL automatically detects <br/> 
        deadlocks and resolves them by aborting one of transactions, <br/>
        allowing the other to complete. </i>
      </td>
    </tr>
    <tr>
      <td>7</td>
      <td>
        <pre>UPDATE 1</pre>
        <i>Transaction completes the query.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>8</td>
      <td>
        <pre>psql> COMMIT;</pre>
        <pre>
psql> SELECT * FROM kids;
<p>
 id | name | age
----+------+-----
  1 | Ann  |   10
  2 | Ben  |    9
  3 | Sam  |    5
</p>
      </pre>
      </td>
      <td></td>
    </tr>
  </tbody>
</table>
<br>

| id      | name | age |
| ----------- | ----------- | ----------- |
|1|Ann|10|
|2|Ben|9|  
|3|Sam|5| 

<br>


### Deadlock on explicit exclusive row-level locks (SELECT FOR UPDATE)
   
   
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
START TRANSACTION
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
START TRANSACTION
        </pre>
      </td>
    </tr>
    <tr>
      <td>3</td>
      <td>
        <pre>
psql> SELECT * FROM kids    
      WHERE name='Ann' FOR UPDATE;
<br>
 id | name | age
----+------+-----
  1 | Ann  |  10
        </pre>
        <i>The <b>client#1</b> acquires an exclusive <br />
          row-level lock on the 'Ann' row.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>4</td>
      <td></td>
      <td>
        <pre>
psql> SELECT * FROM kids    
      WHERE name='Ben' FOR UPDATE;
<br>
 id | name | age
----+------+-----
  2 | Ben  |   9
        </pre>
        <i>The <b>client#2</b> acquires an exclusive<br /> 
          row-level lock on the 'Ben' row.</i>
      </td>
    </tr>
    <tr>
      <td>5</td>
      <td>
        <pre>
psql> UPDATE kids SET age=13    
      WHERE name='Ben';
...
        </pre>
        <i>The <b>client#1</b> attempts to update<br />
          the 'Ben' row and falls into the waiting mode.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>6</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE kids    
      SET age=14 WHERE name='Ann';
<br>
ERROR:  deadlock detected<br />
DETAIL:  Process 42439 waits for ShareLock  <br />  
on transaction 17685; blocked by process 84586.  <br />  
Process 84586 waits for ShareLock    <br />
on transaction 17686; blocked by process 42439.
        </pre>
        <i>The <b>client#2</b> attempts<br />
          to update the 'Ann' row. Postgres detects the deadlock<br />    
          and aborts the query, allowing the <b>client#1</b> to succeed.</i>
      </td>
    </tr>
    <tr>
      <td>7</td>
      <td>
        <pre>
UPDATE 1
<br>
psql> COMMIT;
COMMIT
<br>
psql> SELECT * FROM kids;
<br>
 id | name | age
----+------+-----
  3 | Sam  |   5
  1 | Ann  |  10
  2 | Ben  |  13
        </pre>
      </td>
      <td></td>
    </tr> 
  </tbody>
</table>

