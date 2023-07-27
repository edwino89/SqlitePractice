# SQLITE AS GRAPH DATABASE
Idea is to use sqlite like Neo4J; 
* So nodes and edges
* CRUD, Filter, Aggregate, Views... 

#### SQLITE COMMANDS
* Import SQL into sqlite3 path/to/db < path/to/file.sql
* Format query outputs .mode markdown

## CREATE TABLES
* CREATE TABLE nodes (
id integer primary key autoincrement,
data text);

* CREATE TABLE edges(
id integer unique,
previous_node integer,
next_node integer,
foreign key(previous_node) references nodes(id),
foreign key(next_node) references nodes(id),
primary key(previous_node, next_node, id));

## INSERT DATA
nb Should add properties to field also edges id is manually entered coz sqlite does not auto increment non primary key fields
Also Edges have loops; So no top down manenos

INSERT INTO nodes (data) VALUES ('Bob');
INSERT INTO nodes (data) VALUES ('Hank');
INSERT INTO nodes (data) VALUES ('Jeff');
INSERT INTO nodes (data) VALUES ('Sally');
INSERT INTO nodes (data) VALUES ('Sue');
INSERT INTO nodes (data) VALUES ('Sam');

INSERT INTO edges (id, previous_node, next_node) VALUES (1, 1, 2);
INSERT INTO edges (id, previous_node, next_node) VALUES (2, 1, 3);
INSERT INTO edges (id, previous_node, next_node) VALUES (3, 2, 4);
INSERT INTO edges (id, previous_node, next_node) VALUES (4, 3, 4);
INSERT INTO edges (id, previous_node, next_node) VALUES (5, 4, 5);

INSERT INTO edges (id, previous_node, next_node) VALUES (98, 3, 1);

INSERT INTO edges (id, previous_node, next_node) VALUES (99, 5, 1);

### SHOW BOB'S FRIENDS UP TO ONE LEVELS
with RECURSIVE 
    foff (nxtnode, path, depth) as (
        select edges.next_node, cast(edges.id as text),1 
        from edges
        where edges.previous_node = 1
        UNION ALL
        select edges.next_node, path || "," || edges.id, depth+1
        from edges, foff
        where edges.previous_node = foff.nxtnode and
        not instr(cast(foff.path as text),(select(cast(edges.id as text)))) and
        foff.depth <1
    )
select *
    from nodes 
    join foff on nodes.id = foff.nxtnode
    where nodes.id != 1
    group by nodes.id
    order by foff.path;

### SHOW BOB'S FRIENDS UP TO TWO LEVELS
with RECURSIVE 
    foff (nxtnode, path, depth) as (
        select edges.next_node, cast(edges.id as text),1 
        from edges
        where edges.previous_node = 1
        UNION ALL
        select edges.next_node, path || "," || edges.id, depth+1
        from edges, foff
        where edges.previous_node = foff.nxtnode and
         not instr(cast(foff.path as text),(select(cast(edges.id as text)))) and
         foff.depth <2
    )
select *
    from nodes 
    join foff on nodes.id = foff.nxtnode
    where nodes.id != 1
    group by nodes.id
    order by foff.path;

### SHOW BOB'S FRIENDS UP TO THREE LEVELS
with RECURSIVE 
    foff (nxtnode, path, depth) as (
        select edges.next_node, cast(edges.id as text),1 
        from edges
        where edges.previous_node = 1
        UNION ALL
        select edges.next_node, path || "," || edges.id, depth+1
        from edges, foff
        where edges.previous_node = foff.nxtnode and
         not instr(cast(foff.path as text),(select(cast(edges.id as text)))) and
         foff.depth <3
    )
select *
    from nodes 
    join foff on nodes.id = foff.nxtnode
    where nodes.id != 1
    group by nodes.id
    order by foff.path;

### SHOW JEFFS'S FRIENDS UP TO TWO LEVELS
with RECURSIVE 
    foff (nxtnode, path, depth) as (
        select edges.next_node, cast(edges.id as text),1 
        from edges
        -- Jeff's ID
        where edges.previous_node = 3
        UNION ALL
        select edges.next_node, path || "," || edges.id, depth+1
        from edges, foff
        where edges.previous_node = foff.nxtnode and
        not instr(cast(foff.path as text),(select(cast(edges.id as text)))) and
        -- Max Depth
        foff.depth <2
    )
select *
    from nodes 
    join foff on nodes.id = foff.nxtnode
    -- Jeff's ID
    where nodes.id != 3
    group by nodes.id
    order by foff.path;