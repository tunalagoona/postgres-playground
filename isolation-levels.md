## Transaction Isolation Levels

Database structure:

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

<table>
  <thead>
    <th>Client1</th>
    <th>Client2</th>
  </thead>
  <tbody>
  <tr>
    <td>
      <pre>
psql> START TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION
      </pre>
    </td>
    <td>
      <pre>
psql> START TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION
      </pre>
    </td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>
psql> UPDATE kids SET age=7 WHERE name='Ann';
UPDATE 1
      </pre>
    </td>
  </tr>
  <tr>
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
    <i>The SELECT query doesn't see uncommitted data.</i>
    </td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>
psql> COMMIT;
COMMIT
      </pre>
    </td>
  </tr>
  <tr>
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
    <i>After the data is committed, it becomes visible to the transaction.</i>
    </td>
    <td></td>
  </tr>
  </tbody>
</table>

### Repeatable read

<table>
  <thead>
    <th>Client1</th>
    <th>Client2</th>
  </thead>
  <tbody>
  <tr>
    <td>
      <pre>
psql> START TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION
      </pre>
    </td>
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
    <td></td>
    <td>
      <pre>
psql> UPDATE kids SET age=12 WHERE name='Ben';
UPDATE 1
      </pre>
      <pre>
psql> COMMIT;
COMMIT
      </pre>
    </td>
  </tr>
  <tr>
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
      <i>The transaction only sees data committed</ br>
      before it began. </i>
    </td>
    <td></td>
  </tr>
  </tbody>
</table>
