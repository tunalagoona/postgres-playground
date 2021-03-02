## PostgreSQL DeadLocks

Let's create a table and populate it with values:

>CREATE TABLE locks (  
> id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  
> name VARCHAR,  
> age INTEGER  
>);
>
>INSERT INTO locks (name, age) VALUES ('Ann', 7);   
>INSERT INTO locks (name, age) VALUES ('Ben', 12);
>

Our table should now look like this:

  
| id      | name | age |
| ----------- | ----------- | ----------- |
|1|Ann|7|
|2|Ben|12|   
   
Let's try to update the rows from 2 clients concurrently:

<table>
  <thead>
    <th>Client1</th>
    <th>Client2</th>
  </thead>
  <tbody>
  <tr>
    <td>
      <pre>START TRANSACTION;</pre>
    </td>
    <td>
      <pre>START TRANSACTION;</pre>
    </td>
  </tr>
  <tr>
    <td>
      <pre>UPDATE locks SET age=10 WHERE name='Ann';</pre>
    </td>
    <td>
      <pre>UPDATE locks SET age=13 WHERE name='Ben';</pre>
    </td>
  </tr>
  <tr>
    <td>
      <pre>UPDATE locks SET age=9 WHERE name='Ben';</pre>
      The query stucks in waiting mode.
      An exclusive row-level lock had been acquired when the row was updated by Client2
    </td>
    <td>
      <pre>UPDATE locks SET age=5 WHERE name='Ann';</pre>
      An exclusive row-level lock had been acquired when the row was updated by Client1
      error:
      <pre>
        ERROR:  deadlock detected
        DETAIL:  Process 37184 waits for ShareLock on transaction 17500; blocked by process 37281.
        Process 37281 waits for ShareLock on transaction 17501; blocked by process 37184.
        HINT:  See server log for query details.
        CONTEXT:  while updating tuple (0,3) in relation "locks"
      </pre>
      two (or more) transactions each hold locks that the other wants.
      PostgreSQL automatically detects deadlock situations and resolves them by aborting one of the transactions involved, allowing the other(s) to complete. 
    </td>
  </tr>
  <tr>
    <td>
      <pre>UPDATE 1</pre>
    </td>
    <td></td>
  </tr>
  <tr>
    <td>
      <pre>COMMIT;</pre>
      <pre>
  psql> SELECT * FROM locks;
  <p>
   id | name | age
  ----+------+-----
    1 | Ann  |   10
    2 | Ben  |    9
  </p>
    </pre>
    The lock is held until the transaction commits or rolls back. 
    </td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>COMMIT;</pre>
      <pre>ROLLBACK</pre>
      <pre>
  psql> SELECT * FROM locks;
  <p>
   id | name | age
  ----+------+-----
    1 | Ann  |   10
    2 | Ben  |    9
  </p>
    </pre>
    </td>
  </tr>
  </tbody>
</table>

