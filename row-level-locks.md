
## Row-Level Locks

[Reference](https://www.postgresql.org/docs/9.1/explicit-locking.html)

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
psql> UPDATE weather SET temperature=11   
      WHERE the_date='2020-04-15';   

UPDATE 1
        </pre>
    <i>An exclusive row-level lock for the row </br>
        is automatically acquired by the **client#1** when the row is updated.</i>
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
        <i>The row-level lock acquired by the <b>client#1</b> </br>
          does not affect the select statement issued by the <b>client#2</b>.</i>
      </td>
    </tr>
    <tr>
      <td>5</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE weather SET temperature=22    
      WHERE the_date='2020-04-17';   

  ...
        </pre>
        <i>The row-level lock acquired by the <b>client#1</b> </br>
        does not affect the <b>client#2</b> from writing to a different row.</i>
      </td>
    </tr>
    <tr>
      <td>6</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE weather SET temperature=0    
      WHERE the_date='2020-04-15';   

  ...
        </pre>
        <i>However, the row-level lock acquired by the <b>client#1</b> </br>
        blocks the <b>client#2</b> from writing to the same row.</i>        
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
...
UPDATE 1
        </pre>
        <i>Once the <b>client#1</b> transaction is committed,  
    the lock is released. The <b>client#2</b> acquires the lock and completes the update.</i>
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
|3|2020-04-17|10| 
<br>

### Explicit shared row-level lock (SELECT FOR SHARE)

> "To acquire a shared row-level lock on a row, select the row with SELECT FOR SHARE. A shared lock does not prevent other transactions from acquiring the same shared lock. However, no transaction is allowed to update, delete, or exclusively lock a row on which any other transaction holds a shared lock. Any attempt to do so will block until the shared lock(s) have been released."
   
   
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
psql> SELECT * FROM weather    
      WHERE the_date='2020-04-16' FOR SHARE;
<br> 
 id |  the_date  | temperature
----+------------+-------------
  2 | 2020-04-16 |           5
      </pre>
        <i>The <b>client#1</b> transaction acquires a shared row-level lock on the row.</i>
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
        <i>A shared lock, acquired by the <b>client#1</b>, </br>
        does not prevent the <b>client#2</b> transaction  <br>  
        from acquiring the same shared lock.</i>
      </td>
    </tr> 
    <tr>
      <td>5</td>
      <td></td>
      <td>
        <pre>
psql> UPDATE weather SET temperature=0   
      WHERE the_date='2020-04-16';  
  ...
        </pre>
        <i>The <b>client#2</b> transaction is not allowed to update a row   
          on which the <b>client#1</b> transaction holds a shared lock.</i>
      </td>
    </tr>
    <tr>
      <td>6</td>
      <td>
        <pre>
psql> COMMIT;
COMMIT
        </pre>
        <i>After the <b>client#1</b> transaction is committed,   
          the shared lock on the row is released.</i>
      </td>
      <td></td>
    </tr> 
    <tr>
      <td>7</td>
      <td></td>
      <td>
        <pre>
UPDATE 1
        </pre>
        <i>The <b>ck=lient#2</b> transaction is now allowed </br>
        to acquire exclusive lock and update a row on which </br>
        the <b>client#1</b> transaction holded a shared lock.</i>
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

### Explicit exclusive row-level lock (SELECT FOR UPDATE)

> "To acquire an exclusive row-level lock on a row without actually modifying the row, select the row with SELECT FOR UPDATE. Note that once the row-level lock is acquired, the transaction can update the row multiple times without fear of conflicts."
   
   
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
psql> SELECT * FROM weather   
      WHERE the_date='2020-04-17' FOR UPDATE;
<br>
id  |  the_date  | temperature
----+------------+-------------
  3 | 2020-04-17 |          10
        </pre>
        <i>The <b>client#1</b> transaction acquires an exclusive </br>
        row-level lock on the row without actually modifying it.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>4</td>
      <td></td>
      <td>
        <pre>
psql> SELECT * FROM weather   
      WHERE the_date='2020-04-17' FOR UPDATE;
  ...
        </pre>
        <i>The <b>client#2</b> transaction can't acquire </br>
          an exclusive row-level lock on the row.</i>
      </td>
    </tr>
    <tr>
      <td>5</td>
      <td></td>
      <td>
        <i>Let's rollback the <b>client#1</b> transation </br>
        so we could try to update the row.</i>
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
        <i>The query falls in a waiting mode until </br>
        the <b>client#1</b> transaction releases the lock.</i>
      </td>
    </tr>
    <tr>
      <td>6</td>
      <td>
        <pre>
psql> COMMIT;
COMMIT
      </pre>
        <i>The <b>client#1</b> transaction releases the lock.</i>
      </td>
      <td></td>
    </tr>
    <tr>
      <td>7</td>
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
        <i>The <b>client#2</b> transaction acquires the lock and updates the row.</i>
      </td>
    </tr>
  </tbody>
</table>
