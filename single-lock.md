
## PostgreSQL Single Locks

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

   
### implicit exclusive row-level lock   
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

<br>

| id      | the_date | temperature |
| ----------- | ----------- | ----------- |
|1|2020-04-15|0|
|2|2020-04-16|5|  
|3|2020-04-17|10| 
<br>

### explicit shared row-level lock   
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
psql> SELECT * FROM weather    
      WHERE the_date='2020-04-16' FOR SHARE;
<br> 
 id |  the_date  | temperature
----+------------+-------------
  2 | 2020-04-16 |           5
      </pre>
      <i>The transaction acquires a shared row-level lock on a row.</i>
    </td>
    <td></td>
  </tr> 
  <tr>
    <td></td>
    <td>
      <pre>
psql> SELECT * FROM weather    
      WHERE the_date='2020-04-16' FOR SHARE;
<br>
 id |  the_date  | temperature
----+------------+-------------
  2 | 2020-04-16 |           5
      </pre>
      <i>A shared lock does not prevent the second transaction    
        from acquiring the same shared lock.</i>
    </td>
  </tr> 
  <tr>
    <td></td>
    <td>
      <pre>
psql> UPDATE weather SET temperature=0   
      WHERE the_date='2020-04-16';  
...
      </pre>
      <i>The second transaction is not allowed to update a row   
        on which the first transaction holds a shared lock.</i>
    </td>
  </tr>
  <tr>
    <td>
      <pre>
psql> COMMIT;
COMMIT
      </pre>
      <i>After the first transaction is committed,   
        the shared lock on the row is released.</i>
    </td>
    <td></td>
  </tr> 
  <tr>
    <td></td>
    <td>
      <pre>
UPDATE 1
      </pre>
      <i>The second transaction is now allowed to acquire    
        exclusive lock and update a row on which the first   
        transaction holded a shared lock.</i>
    </td>
  </tr> 
  </tbody>
</table>

<br>

| id      | the_date | temperature |
| ----------- | ----------- | ----------- |
|1|2020-04-15|0|
|2|2020-04-16|0|  
|3|2020-04-17|10| 
<br>

### explicit exclusive row-level lock 

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
psql> SELECT * FROM weather   
      WHERE the_date='2020-04-17' FOR UPDATE;
<br>
id |  the_date  | temperature
----+------------+-------------
  3 | 2020-04-17 |          10
      </pre>
      <i>The first transaction acquires an exclusive row-level lock   
        on a row without actually modifying the row.</i>
    </td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>
psql> SELECT * FROM weather   
      WHERE the_date='2020-04-17' FOR UPDATE;
...
      </pre>
      <i>The second transaction can't acquire 
        an exclusive row-level lock on the row.</i>
    </td>
  </tr>
  <tr>
    <td></td>
    <td>
      <i>Let's rollback the second transation so we could try to update the row.</i>
      <pre>
psql> ROLLBACK;
ROLLBACK
<br>
psql> START TRANSACTION;
START TRANSACTION
<br>
psql> UPDATE weather SET temperature=0   
      WHERE the_date='2020-04-17';
...
      </pre>
      <i>The query falls in a waiting mode until the first transaction releases the lock.</i>
    </td>
  </tr>
  <tr>
    <td>
      <pre>
psql> COMMIT;
COMMIT
      </pre>
      <i>The first transaction releases the lock.</i>
    </td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td>
      <pre>
UPDATE 1
<br>
psql> COMMIT;
COMMIT

psql> SELECT * FROM weather;
<br>
 id |  the_date  | temperature
----+------------+-------------
  1 | 2020-04-15 |           0
  2 | 2020-04-16 |           0
  3 | 2020-04-17 |           0
      </pre>
      <i>The second transaction acquires the lock and updates the row.</i>
    </td>
  </tr>
  </tbody>
</table>
