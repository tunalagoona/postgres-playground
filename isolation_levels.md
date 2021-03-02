Let's create a table and populate it with some values.

>CREATE TABLE locks (  
>id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, 
>name VARCHAR, 
>age INTEGER  
>);
>
>INSERT INTO locks (name, age) VALUES ('Ann', 6);
>INSERT INTO locks (name, age) VALUES ('Ben', 8);
>

Our table now should look like this:

  
| id      | name | age |
| ----------- | ----------- | ----------- |
|1|Ann|6|
|2|Ben|8|

Let's initiate 2 transactions and see how isolation levels differ from each other:

**READ COMMITTED**

| Tx1      | Tx2 |
| ----------- | ----------- |
| START TRANSACTION ISOLATION LEVEL READ COMMITTED; | START TRANSACTION;     | 
|      | |
|      | UPDATE locks SET age=7 WHERE name='Ann';|
|      | |
| SELECT * FROM locks; | |
|id name age|      | 
|1  Ann  6| |
|2  Ben  8| |
|Doesn't see uncommitted changes||
| | |
| | COMMIT;|
| | |
| SELECT * FROM locks; | |
|id name age|      | 
|2  Ben  8| |
|1  Ann  7| |
|After the changes committed tjey become visible for the transaction||
|||

**REPEATABLE READ**

| Tx1      | Tx2 |
| ----------- | ----------- |
|||
|||
|||
|||
|||
|||
|||
|||
|||
