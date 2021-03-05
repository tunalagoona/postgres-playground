
## DeadLocks

[Reference](https://www.postgresql.org/docs/9.1/explicit-locking.html)

### Database structure:

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
   

### Deadlock on implicit exclusive row-level locks 
   
   
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
psql> UPDATE children SET age=10 WHERE name='Ann';
UPDATE 1
        </pre>
        <i>
          An exclusive row-level lock had been acquired by the <b>client#1</b>.<br />
          The lock is held until the transaction is committed or rolled back.
        </i>
      </td>
      <td> </td>
    </tr>
    <tr>
      <td>4</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE children SET age=13 WHERE name='Ben';
UPDATE 1
        </pre>
        <i>
        An exclusive row-level lock had been acquired by the <b>client#2</b>.<br />
        The lock is held until the transaction is committed or rolled back.
        </i>
      </td>
    </tr>
    <tr>
      <td>5</td>
      <td>
        <pre>
psql> UPDATE children SET age=9 WHERE name='Ben';
...
        </pre>
        <i>The query stucks in a waiting mode. <br />
        <b>client#2</b> transaction holds the lock <br />
        that the current transaction wants.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>6</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE children SET age=5 WHERE name='Ann';

ERROR:  deadlock detected
DETAIL:  Process 37184 waits for ShareLock <br />
on transaction 17500; blocked by process 37281.<br />
Process 37281 waits for ShareLock on transaction 17501; <br />
blocked by process 37184.
        </pre>
        <i>Two transactions each hold locks that the other wants.<br /> 
        PostgreSQL automatically detects deadlock situations <br />
        and resolves them by aborting one of transactions, <br />
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
      <td>9</td>
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
<br>

| id      | name | age |
| ----------- | ----------- | ----------- |
|1|Ann|10|
|2|Ben|9|  
|3|Sam|5| 

<br>


### Deadlock on explicit exclusive row-level locks    
   
   
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
psql> SELECT * FROM children    
      WHERE name='Ann' FOR UPDATE;
<br>
 id | name | age
----+------+-----
  1 | Ann  |  10
        </pre>
        <i>The <b>client#1</b> transaction acquires exclusive <br />
          row-level lock on the 'Ann' row.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>4</td>
      <td></td>
      <td>
        <pre>
psql> SELECT * FROM children    
      WHERE name='Ben' FOR UPDATE;
<br>
 id | name | age
----+------+-----
  2 | Ben  |   9
        </pre>
        <i>The <b>client#2</b> transaction acquires exclusive<br /> 
          row-level lock on the 'Ben' row.</i>
      </td>
    </tr>
    <tr>
      <td>5</td>
      <td>
        <pre>
psql> UPDATE children SET age=13    
      WHERE name='Ben';
...
        </pre>
        <i>The <b>client#1</b> transaction attempts to update<br />
          the 'Ben' row and falls in a waiting mode.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>6</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE children    
      SET age=14 WHERE name='Ann';
<br>
ERROR:  deadlock detected<br />
DETAIL:  Process 42439 waits for ShareLock  <br />  
on transaction 17685; blocked by process 84586.  <br />  
Process 84586 waits for ShareLock    <br />
on transaction 17686; blocked by process 42439.
        </pre>
        <i>The <b>client#2</b> transaction attempts<br />
          to update the 'Ann' row. Postgres detects deadlock<br />    
          and aborts the query, allowing the <b>client#1</b> transaction<br />     
          to succeed.</i>
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
psql> SELECT * FROM children;
<br>
 id | name | age
----+------+-----
  3 | Sam  |   5
  1 | Ann  |  10
  2 | Ben  |  13
        </pre>
        <i>The <b>client#1</b> transaction updates the 'Ben' row<br />  
          and releases the lock upon commit.</i>
      </td>
      <td></td>
    </tr> 
  </tbody>
</table>

