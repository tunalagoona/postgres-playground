
## PostgreSQL Multi Locks

Let's create a table and populate it with values:

```
psql> CREATE TABLE children (  
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  
  name VARCHAR,  
  age INTEGER  
);

psql> INSERT INTO children (name, age) VALUES ('Ann', 7);   
psql> INSERT INTO children (name, age) VALUES ('Ben', 12);
psql> INSERT INTO children (name, age) VALUES ('Sam', 5);
```

  
| id      | name | age |
| ----------- | ----------- | ----------- |
|1|Ann|7|
|2|Ben|12|  
|3|Sam|5| 
   
Let's try to update the same rows from 2 clients concurrently.

## Deadlock on implicit exclusive row-level locks 

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
START TRANSACTION
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
      </pre>
    </td>
  </tr>
  <tr>
    <td>
      <pre>
psql> UPDATE children SET age=10 WHERE name='Ann';
UPDATE 1
      </pre>
      <i>
        An exclusive row-level lock had been acquired.<br />
        The lock is held until the transaction is committed or rolled back.
      </i>
    </td>
    <td> </td>
  </tr>
  <tr>
    <td> </td>
    <td>
      <pre>
psql> UPDATE children SET age=13 WHERE name='Ben';
UPDATE 1
      </pre>
      <i>
      An exclusive row-level lock had been acquired.<br />
      The lock is held until the transaction is committed or rolled back.
      </i>
    </td>
  </tr>
  <tr>
    <td>
      <pre>
psql> UPDATE children SET age=9 WHERE name='Ben';
...
      </pre>
      <i>The query stucks in a waiting mode. <br />
      Another transaction holds the lock that current transaction wants.</i>
    </td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>
psql> UPDATE children SET age=5 WHERE name='Ann';

ERROR:  deadlock detected
DETAIL:  Process 37184 waits for ShareLock <br />on transaction 17500; blocked by process 37281.
Process 37281 waits for ShareLock on transaction 17501; <br />blocked by process 37184.
      </pre>
      <i>Two transactions each hold locks that the other wants.<br /> 
      PostgreSQL automatically detects deadlock situations <br />
      and resolves them by aborting one of transactions, <br />
      allowing the other to complete. </i>
    </td>
  </tr>
  <tr>
    <td>
      <pre>UPDATE 1</pre>
      <i>Transaction completes the query.</i>
    </td>
    <td></td>
  </tr>
  <tr>
    <td>
      <pre>psql> COMMIT;</pre>
      <pre>
psql> SELECT * FROM children;
<p>
 id | name | age
----+------+-----
  1 | Ann  |   10
  2 | Ben  |    9
  3 | Sam  |    5
</p>
    </pre>
    <i>The changes are committed.</i>
    </td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>
psql>COMMIT;
ROLLBACK</pre>
      <pre>
psql> SELECT * FROM children;
<p>
 id | name | age
----+------+-----
  1 | Ann  |   10
  2 | Ben  |    9
  3 | Sam  |    5
</p>
    </pre>
      <i>The changes are rolled back.</i>
    </td>
  </tr>
  </tbody>
</table>
