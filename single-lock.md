
## PostgreSQL Locks

Let's create a table and populate it with values:

```
psql> CREATE TABLE weather (  
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,  
  the_date DATE,
  temperature INTEGER 
);
CREATE TABLE

psql> INSERT INTO weather (the_date, temperature) VALUES ('2020-04-15', 8);   
psql> INSERT INTO weather (the_date, temperature) VALUES ('2020-04-16', 5);
psql> INSERT INTO weather (the_date, temperature) VALUES ('2020-04-17', 10);
```

  
| id      | the_date | temperature |
| ----------- | ----------- | ----------- |
|1|2020-04-15|8|
|2|2020-04-16|5|  
|3|2020-04-17|10| 

   
   
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

  </tbody>
</table>
