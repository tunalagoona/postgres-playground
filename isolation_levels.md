## PostgreSQL Isolation Levels

Let's create a table and populate it with values:

>CREATE TABLE locks (  
> id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  
> name VARCHAR,  
> age INTEGER  
>);
>
>INSERT INTO locks (name, age) VALUES ('Ann', 6);   
>INSERT INTO locks (name, age) VALUES ('Ben', 8);
>

Our table should now look like this:

  
| id      | name | age |
| ----------- | ----------- | ----------- |
|1|Ann|6|
|2|Ben|8|   
   
Let's initiate 2 transactions and see first how **Read Committed** Isolation level works:

### Read Committed

<table>
  <thead>
    <th>Client1</th>
    <th>Client2</th>
  </thead>
  <tbody>
  <tr>
    <td>
      <pre>START TRANSACTION ISOLATION LEVEL READ COMMITTED;</pre>
    </td>
    <td>
      <pre>START TRANSACTION;</pre>
    </td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>UPDATE locks SET age=7 WHERE name='Ann';</pre>
    </td>
  </tr>
  <tr>
    <td>
      <pre>
  psql> SELECT * FROM locks;;

  <p>
   id | name | age
  ----+------+-----
    1 | Ann  |   6
    2 | Ben  |   8
  </p>
    </pre>
    The SELECT query never sees uncommitted data.
    </td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>COMMIT;</pre>
    </td>
  </tr>
  <tr>
    <td>
      <pre>
  psql> SELECT * FROM locks;;

  <p>
   id | name | age
  ----+------+-----
    1 | Ann  |   7
    2 | Ben  |   8
  </p>
    </pre>
    The query sees the data committed before the query began.
    </td>
    <td></td>
  </tr>
  </tbody>
</table>

Now let's see how **Repeatable Read** Isolation works:

### Repeatable read

<table>
  <thead>
    <th>Client1</th>
    <th>Client2</th>
  </thead>
  <tbody>
  <tr>
    <td>
      <pre>START TRANSACTION ISOLATION LEVEL REPEATABLE READ;</pre>
    </td>
    <td>
      <pre>START TRANSACTION;</pre>
    </td>
  </tr>
  <tr>
    <td>
        <pre>
  psql> SELECT * FROM locks;

  <p>
   id | name | age
  ----+------+-----
    1 | Ann  |   7
    2 | Ben  |   8
  </p>
    </pre>
    </td>
    <td></td>
  </tr>
    <td></td>
    <td>
      <pre>UPDATE locks SET age=12 WHERE name='Ben';</pre>
      <pre>COMMIT;</pre>
    </td>
  </tr>
  <tr>
    <td>
      <pre>
  psql> SELECT * FROM locks;

  <p>
   id | name | age
  ----+------+-----
    1 | Ann  |   7
    2 | Ben  |   8
  </p>
    </pre>
      The Repeatable Read isolation level only sees data committed before the transaction began.
    </td>
    <td></td>
  </tr>
  </tbody>
</table>
