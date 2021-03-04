## PostgreSQL DeadLock Prevention

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
   
Let's try to update the same rows from 2 clients concurrently:

<table>
  <thead>
    <th>Client1</th>
    <th>Client2</th>
  </thead>
  <tbody>
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
    
