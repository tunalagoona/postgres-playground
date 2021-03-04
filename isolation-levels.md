## Transaction Isolation

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
    <th>client#1</th>
    <th>client#2</th>
  </thead>
  <tbody>
  <tr>
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
    <i>The SELECT query can not see the uncommitted changes made by the <b>client#2</b>.</i>
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
    <i>Once the changes are committed by the <b>client#2</b>, they become visible to the <b>client#1</b> transaction.</i>
    </td>
    <td></td>
  </tr>
  </tbody>
</table>

### Repeatable Read

<table>
  <thead>
    <th>client#1</th>
    <th>client#2</th>
  </thead>
  <tbody>
  <tr>
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
