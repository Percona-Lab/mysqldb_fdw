Usage
-----

The following parameters can be set on a MySQL foreign server object:

  * `host`: Address or hostname of the MySQL server. Defaults to `127.0.0.1`
  * `port`: Port number of the MySQL server. Defaults to `3306`
  * `secure_auth`: Enable or disable secure authentication. Default is `true`

The following parameters can be set on a MySQL foreign table object:

  * `dbname`: Name of the MySQL database to query. This is a mandatory option.
  * `table_name`: Name of the MySQL table, default is the same as foreign table.

The following parameters need to supplied while creating user mapping.

  * `username`: Username to use when connecting to MySQL.
  * `password`: Password to authenticate to the MySQL server with.


-- load extension first time after install

    CREATE EXTENSION mysql_fdw;

-- create server object

    CREATE SERVER mysql_server
         FOREIGN DATA WRAPPER mysql_fdw
         OPTIONS (host '127.0.0.1', port '3306');

-- create user mapping

    CREATE USER MAPPING FOR postgres
	SERVER mysql_server
	OPTIONS (username 'foo', password 'bar');

-- create foreign table

    CREATE FOREIGN TABLE warehouse(
         warehouse_id int,
         warehouse_name text,
         warehouse_created datetime)
    SERVER mysql_server
         OPTIONS (dbname 'db', table_name 'warehouse');


-- insert new rows in table

    INSERT INTO warehouse values (1, 'UPS', sysdate());
    INSERT INTO warehouse values (2, 'TV', sysdate());
    INSERT INTO warehouse values (3, 'Table', sysdate());


-- select from table

    SELECT * FROM warehouse;

    warehouse_id | warehouse_name | warehouse_created  

    --------------+----------------+--------------------

            1 | UPS            | 29-SEP-14 23:33:46

            2 | TV             | 29-SEP-14 23:34:25

            3 | Table          | 29-SEP-14 23:33:49


-- delete row from table

    DELETE FROM warehouse where warehouse_id = 3;


-- update a row of table

    UPDATE warehouse set warehouse_name = 'UPS_NEW' where warehouse_id = 1;


-- explain a table

    EXPLAIN SELECT warehouse_id, warehouse_name FROM warehouse WHERE warehouse_name LIKE 'TV' limit 1;

                                       QUERY PLAN                                                   
    Limit  (cost=10.00..11.00 rows=1 width=36)
    ->  Foreign Scan on warehouse  (cost=10.00..13.00 rows=3 width=36)
         Local server startup cost: 10
         Remote query: SELECT warehouse_id, warehouse_name FROM db.warehouse WHERE ((warehouse_name like 'TV'))
 Planning time: 0.564 ms
(5 rows)

