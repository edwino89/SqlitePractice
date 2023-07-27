# SQLITE AS DOCUMENT DATABASE
Idea is to use sqlite like Mongo DB; 
* So no joins and 
* embed documents
* CRUD, Filter, Aggregate, Views... 

## Common Commands
* To Create a DB - sqlite3 jsondb.db
* To list attached database(s) - .database
* To show schema - .schema
* To Tun on Headers - .headers ON;
* Format - .mode markup
* To export as Excel = .excel ,before running your command
* As to alias names of select, colums, functions... etc

## Create a Table
CREATE TABLE users(id INTEGER PRIMARY KEY AUTOINCREMENT, data JSON);

## insert into table

### insert blank info
insert into users (data) values (json_insert());

### insert info

insert into users (data) values (json_insert('{"name":"John", "age":25,"gender":"Male", "hobbies":[{"indoors":["stamp collection","reading"]}, {"outdoors":["swiming","bike riding"]}]}'));

insert into users (data) values (json_insert('{"name":"Alice", "age":25,"gender":"Female", "hobbies":[{"indoors":["baking","reading"]}, {"outdoors":["swiming","hiking"]}]}'));

insert into users (data) values (json_insert('{"name":"Bob", "age":18,"gender":"Male", "hobbies":[{"indoors":["video games"]}, {"outdoors":["rugby","hiking"]}]}'));

insert into users (data) values (json_insert('{"name":"Jane", "age":22,"gender":"Female", "hobbies":[{"indoors":["knitting","fashion"]}, {"outdoors":["travelling"]}]}'));

insert into users (data) values (json_insert('{"name":"Jeniffer", "age":22,"gender":"Female", "hobbies":[{"indoors":["knitting","fashion"]}, {"outdoors":["travelling", "rugby"]}]}'));


## read data from table
sqlite> select id, json_extract(data, '$.name') as Name, json_extract(data, '$.age') as Age, json_extract(data, '$.gender') as Gender,json_extract(data, '$.hobbies') as Hobbies from users;

## update info on table
* Can't update single key in a json field. Have to update all. So copy and overwrite with the new info on top

update users set data = (json_replace('{"name":"Tom","age":29,"gender":"Male", "hobbies":[{"indoors":["video games", "reading"]}]}')) where id=13;

#### NB: you can use json_replace or json_set

update users set data = (json_replace('{"name":"Tom","age":29,"gender":"Male", "hobbies":[{"indoors":["video games", "reading"]},{"outdoors":["soccer", "rugby"]} ]}')) where id=13;

update users set data = (json_set('{"name":"Tom","age":26,"gender":"Male", "hobbies":[{"indoors":["video games", "reading"]},{"outdoors":["soccer", "rugby"]} ]}')) where id=13;

## filter data from table

#### json_tree create a mega table with certain columns that you can use to filter the data using multiple where conditions eg key, value, path ...

* id|data|key|value|type|atom|id|parent|fullkey|path


##### deconstructs tree of data with focus on hobbies
select * from users,json_tree(data,'$.hobbies'); 

select * from users,json_tree(data,'$.hobbies') where json_tree.key LIKE '%outdoors%'; 

#### users with swimming as an outdoor hobby
select * from users,json_tree(data,'$.hobbies') where json_tree.key LIKE '%outdoors%' and json_tree.value LIKE '%swim%'; 

#### return certain columns
select users.id,key,value from users,json_tree(data,'$.hobbies') where json_tree.key LIKE '%outdoors%' and json_tree.value LIKE '%swim%';

#### Other filters
select * from users where json_extract(data, '$.gender') = 'Male';
select * from users where json_extract(data, '$.age') < 20;
select json_extract(data, '$.hobbies') from users;

select * from (select * from users, json_each(json_extract(users.data,'$.hobbies')) where json_valid(users.data) and json_each.value LIKE '%rugby%') where json_extract(data,'$.age') < 23;

select json_extract(data , '$.name') from (select * from (select * from users, json_each(json_extract(users.data,'$.hobbies')) where json_valid(users.data) and json_each.value LIKE '%rugby%') where json_extract(data,'$.age')< 23);

select json_extract(data, '$.name'),value from users, json_tree(users.data, '$.hobbies') where json_tree.key LIKE '%outdoors%';

select json_extract(data, '$.name'),value from users, json_tree(users.data, '$.hobbies') where json_tree.key LIKE '%outdoors%' and json_tree.value LIKE '%swim%';

select * from (select json_extract(data, '$.name') as NAME ,value as OUTDOOR_HOBBIES from users, json_tree(users.data, '$.hobbies') where json_tree.key LIKE '%outdoors%' and json_tree.value LIKE '%swim%') AS new_data;

## aggregate
select avg(AGE) as AVERAGE_AGE, COUNT(AGE) AS COUNT_AGE from (select id,json_extract(data, '$.age') as AGE from users );

select avg(AGE) as AVERAGE_AGE, COUNT(AGE) AS COUNT_AGE, SUM(AGE) AS SUM_AGE from (select id,json_extract(data, '$.age') as AGE from users );

## delete
delete from users where id = 5;

## Create Views

Create DB
Create Table

Aggregate Material
eg average age, median age


