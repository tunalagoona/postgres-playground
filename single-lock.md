
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
  <tr>
    <td>
      <pre>
psql> UPDATE weather SET temperature=11   
      WHERE the_date='2020-04-15';   
      
UPDATE 1
      </pre>
  <i>An exclusive row-level lock on the row </br>
      is automatically acquired when the row is updated.</i>
    </td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>
psql> SELECT * FROM weather    
      WHERE the_date='2020-04-15';   
<br>
id |  the_date  | temperature
----+------------+-------------
1 | 2020-04-15 |           8
    </pre>
      <i>Row-level lock doesn't affect data querying.</i>
    </td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>
psql> UPDATE weather SET temperature=0    
      WHERE the_date='2020-04-15';   

...
      </pre>
      <i>Row-level lock blocks the writer to the same row.</i>
    </td>
  </tr>
  <tr>
    <td>
      <pre>
psql> COMMIT;   

COMMIT
      </pre>
    </td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>
psql> UPDATE weather SET temperature=0    
      WHERE the_date='2020-04-15';

UPDATE 1
      </pre>
      <i>After the first transaction is committed,  
      the lock is released. Another writer can acquire the lock.</i>
    </td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>
psql> SELECT * FROM weather;   
<br>
id |  the_date  | temperature
----+------------+-------------
2 | 2020-04-16 |           5
3 | 2020-04-17 |          10
1 | 2020-04-15 |           0
      </pre>
    </td>
  </tr>
  
  </tbody>
</table>
