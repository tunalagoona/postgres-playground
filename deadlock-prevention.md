## PostgreSQL DeadLock Prevention

A deadlock situation on a resource can arise if and only if all of the following conditions hold simultaneously in a system(Coffman conditions):
- Mutual exclusion
- Hold and wait or resource holding
- No preemption
- Circular wait

Most approaches work by preventing one of the four Coffman conditions from occurring. 
In this case study we will analyse prevention of two condotions - **the hold and wait** and **circular wait**. 
Prevention of hold and wait can be gained by requiring processes to request all the resources they will need before starting up.
Approaches that avoid circular waits include, for instance, using a hierarchy to determine a partial ordering of resources. 

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
    
