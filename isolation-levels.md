## Transaction Isolation

[Postgres Reference](https://www.postgresql.org/docs/current/transaction-iso.html)

### Database Structure

```
psql> CREATE TABLE kids (  
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  
  name VARCHAR,  
  age INTEGER  
);

psql> INSERT INTO kids (name, age) VALUES ('Ann', 6);   
psql> INSERT INTO kids (name, age) VALUES ('Ben', 8);
psql> INSERT INTO kids (name, age) VALUES ('Sam', 5);
```
  
| id      | name | age |
| ----------- | ----------- | ----------- |
|1|Ann|6|
|2|Ben|8|  
|3|Sam|5| 


### Read Committed

> "When a transaction uses this isolation level, a SELECT query (without a FOR UPDATE/SHARE clause) sees only data committed before the query began; it never sees either uncommitted data or changes committed during query execution by concurrent transactions. (...) two successive SELECT commands can see different data, even though they are within a single transaction, if other transactions commit changes after the first SELECT starts and before the second SELECT starts."

<table>
  <thead>
    <th>#</th>
    <th>client#1</th>
    <th>client#2</th>
  </thead>
  <tbody>
  <tr>
    <td>1</td>
    <td>
      <pre>
psql> START TRANSACTION ISOLATION LEVEL 
      READ COMMITTED;
START TRANSACTION
      </pre>
    </td>
    <td>
      <pre>
psql> START TRANSACTION ISOLATION LEVEL 
      READ COMMITTED;
START TRANSACTION
      </pre>
    </td>
  </tr>
  <tr>
    <td>2</td>
    <td></td>
    <td>
      <pre>
psql> UPDATE kids SET age=7 WHERE name='Ann';
UPDATE 1
      </pre>
    </td>
  </tr>
  <tr>
    <td>3</td>
    <td>
      <pre>
psql> SELECT * FROM kids;

<p>
 id | name | age
----+------+-----
  1 | Ann  |   6
  2 | Ben  |   8
  3 | Sam  |   5
</p>
    </pre>
    <i>The SELECT query can not see the uncommitted changes made by the <b>client#2</b>.</i>
    </td>
    <td></td>
  </tr>
  <tr>
    <td>4</td>
    <td></td>
    <td>
      <pre>
psql> COMMIT;
COMMIT
      </pre>
    </td>
  </tr>
  <tr>
    <td>5</td>
    <td>
      <pre>
psql> SELECT * FROM kids;

<p>
 id | name | age
----+------+-----
  1 | Ann  |   7
  2 | Ben  |   8
  3 | Sam  |   5
</p>
    </pre>
    <i>Once the changes are committed by the <b>client#2</b>, they become visible to the <b>client#1</b> transaction.</i>
    </td>
    <td></td>
  </tr>
  </tbody>
</table>

### Repeatable Read

> "The Repeatable Read isolation level only sees data committed before the transaction began; it never sees either uncommitted data or changes committed during transaction execution by concurrent transactions. (...) a query in a repeatable read transaction sees a snapshot as of the start of the first non-transaction-control statement in the transaction, not as of the start of the current statement within the transaction."

<table>
  <thead>
    <th>#</th>
    <th>client#1</th>
    <th>client#2</th>
  </thead>
  <tbody>
  <tr>
    <td>1</td>
    <td>
      <pre>
psql> START TRANSACTION ISOLATION LEVEL
      REPEATABLE READ;
START TRANSACTION
      </pre>
    </td>
    <td>
      <pre>
psql> START TRANSACTION ISOLATION LEVEL
      READ COMMITTED;
START TRANSACTION
      </pre>
    </td>
  </tr>
  <tr>
    <td>2</td>
    <td>
        <pre>
psql> SELECT * FROM kids;

<p>
 id | name | age
----+------+-----
  1 | Ann  |   7
  2 | Ben  |   8
  3 | Sam  |   5
</p>
    </pre>
    </td>
    <td></td>
  </tr>
    <td>3</td>
    <td></td>
    <td>
      <pre>
psql> UPDATE kids SET age=12 WHERE name='Ben';
UPDATE 1
psql> COMMIT;
COMMIT
      </pre>
    </td>
  </tr>
  <tr>
    <td>4</td>
    <td>
      <pre>
psql> SELECT * FROM kids;

<p>
 id | name | age
----+------+-----
  1 | Ann  |   7
  2 | Ben  |   8
  3 | Sam  |   5
</p>
    </pre>
      <i>While the <b>client#2</b> has already committed the changes, <b>client#1</b> uses the snapshot created at the step 2 which does not include those changes.</i>
    </td>
    <td></td>
  </tr>
  </tbody>
</table>
