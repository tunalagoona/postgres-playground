
## Row-Level Locks

[Postgres Reference](https://www.postgresql.org/docs/9.1/explicit-locking.html)

### Database Structure

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

   
### Implicit exclusive row-level lock (ordinary UPDATE)   

> "(...) there are row-level locks, which can be exclusive or shared locks. An exclusive row-level lock on a specific row is automatically acquired when the row is updated or deleted. The lock is held until the transaction commits or rolls back, just like table-level locks. Row-level locks do not affect data querying; they block only writers to the same row."
   
   
<table>
  <thead>
    <th>#</th>
    <th>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Client#1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </th>
    <th>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Client#2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </th>
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
psql> UPDATE weather SET temperature=11   
      WHERE the_date='2020-04-15';   
UPDATE 1
        </pre>
      <i>An exclusive row-level lock for the row </br>
      is automatically acquired by **Client#1** </br>
      when the row is updated.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>4</td>
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
        <i>The row-level lock acquired by <b>Client#1</b> </br>
          does not affect the select statement </br>
          issued by <b>Client#2</b>.</i>
      </td>
    </tr>
    <tr>
      <td>5</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE weather SET temperature=22  
      WHERE the_date='2020-04-17';
UPDATE 1
        </pre>
        <i>The row-level lock acquired by <b>Client#1</b> </br>
        does not affect <b>Client#2</b> from writing to </br>
        a different row.</i>
      </td>
    </tr>
    <tr>
      <td>6</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE weather SET temperature=0    
      WHERE the_date='2020-04-15';<br>   
...
        </pre>
        <i>However, the row-level lock acquired</br>
        by <b>Client#1</b> blocks <b>Client#2</b> </br>
        from writing to the same row.</i>        
      </td>
    </tr>
    <tr>
      <td>7</td>
      <td>
        <pre>
psql> COMMIT;   
COMMIT
        </pre>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>8</td>
      <td></td>
      <td>
        <pre>
...<br>
UPDATE 1
        </pre>
        <i>Once <b>Client#1</b> transaction is committed,</br>
        the lock is released. <b>Client#2</b> acquires the lock </br>
        and completes the update.</i>
      </td>
    </tr>
    <tr>
      <td>9</td>
      <td></td>
      <td>
        <pre>
psql> SELECT * FROM weather;   
<br>
 id |  the_date  | temperature
----+------------+-------------
  2 | 2020-04-16 |           5
  3 | 2020-04-17 |          22
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
|3|2020-04-17|22| 
<br>

### Explicit shared row-level lock (SELECT FOR SHARE)

> "To acquire a shared row-level lock on a row, select the row with SELECT FOR SHARE. A shared lock does not prevent other transactions from acquiring the same shared lock. However, no transaction is allowed to update, delete, or exclusively lock a row on which any other transaction holds a shared lock. Any attempt to do so will block until the shared lock(s) have been released."
   
   
<table>
  <thead>
    <th>#</th>
    <th>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Client#1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </th>
    <th>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Client#2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </th>
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
psql> SELECT * FROM weather    
      WHERE the_date='2020-04-16' FOR SHARE;
<br> 
 id |  the_date  | temperature
----+------------+-------------
  2 | 2020-04-16 |           5
      </pre>
        <i><b>Client#1</b> acquires a shared row-level lock on the row.</i>
      </td>
      <td></td>
    </tr> 
    <tr>
      <td>4</td>
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
        <i>A shared lock, acquired by <b>Client#1</b>, </br>
        does not prevent <b>Client#2</b> <br>  
        from acquiring the same shared lock.</i>
      </td>
    </tr> 
    <tr>
      <td>5</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE weather SET temperature=0   
      WHERE the_date='2020-04-16'; <br> 
  ...
        </pre>
        <i><b>Client#2</b> is not allowed to update a row   
          on which <b>Client#1</b> holds a shared lock.</i>
      </td>
    </tr>
    <tr>
      <td>6</td>
      <td>
        <pre>
psql> COMMIT;
COMMIT
        </pre>
        <i>Once <b>Client#1</b> transaction is committed,   
          the shared lock on the row is released.</i>
      </td>
      <td></td>
    </tr> 
    <tr>
      <td>7</td>
      <td></td>
      <td>
        <pre>
...<br>
UPDATE 1
        </pre>
        <i><b>Client#2</b> is now allowed </br>
        to acquire an exclusive lock and update the row.</i>
      </td>
    </tr> 
  </tbody>
</table>

<br>

| id      | the_date | temperature |
| ----------- | ----------- | ----------- |
|1|2020-04-15|0|
|2|2020-04-16|0|  
|3|2020-04-17|22| 
<br>

### Explicit exclusive row-level lock (SELECT FOR UPDATE)

> "To acquire an exclusive row-level lock on a row without actually modifying the row, select the row with SELECT FOR UPDATE. Note that once the row-level lock is acquired, the transaction can update the row multiple times without fear of conflicts."
   
   
<table>
  <thead>
    <th>#</th>
    <th>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Client#1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </th>
    <th>       &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Client#2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </th>
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
psql> SELECT * FROM weather   
      WHERE the_date='2020-04-17' FOR UPDATE;
<br>
id  |  the_date  | temperature
----+------------+-------------
  3 | 2020-04-17 |          22
        </pre>
        <i><b>Client#1</b> acquires an exclusive </br>
        row-level lock on the row without actually modifying it.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>4</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE weather SET temperature=0
      WHERE the_date='2020-04-17';<br>
  ...
        </pre>
        <i><b>Client#2</b> has to wait to acquire </br>
          an exclusive (implicit or explicit) row-level lock on the row.</i>
      </td>
    </tr>
    <tr>
      <td>5</td>
      <td>
        <pre>
psql> COMMIT;
COMMIT
      </pre>
        <i><b>Client#1</b> releases the lock.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>6</td>
      <td></td>
      <td>
        <pre>
...<br>
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
        <i><b>Client#2</b> acquires the lock and updates the row.</i>
      </td>
    </tr>
  </tbody>
</table>
