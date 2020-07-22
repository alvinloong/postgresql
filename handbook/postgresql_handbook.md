# [The SQL Language](https://www.postgresql.org/docs/current/sql.html)

## [Data Types](https://www.postgresql.org/docs/current/datatype.html)

### [Arrays](https://www.postgresql.org/docs/current/arrays.html)

#### [Declaration of Array Types](https://www.postgresql.org/docs/current/arrays.html#ARRAYS-DECLARATION)

```
CREATE TABLE sal_emp (
    name            text,
    pay_by_quarter  integer[],
    schedule        text[][]
);
CREATE TABLE tictactoe (
    squares   integer[3][3]
);
pay_by_quarter  integer ARRAY[4],
pay_by_quarter  integer ARRAY,
```

#### [Array Value Input](https://www.postgresql.org/docs/current/arrays.html#ARRAYS-INPUT)

```
INSERT INTO sal_emp
    VALUES ('Bill',
    '{10000, 10000, 10000, 10000}',
    '{{"meeting", "lunch"}, {"training", "presentation"}}');
INSERT INTO sal_emp
    VALUES ('Carol',
    '{20000, 25000, 25000, 25000}',
    '{{"breakfast", "consulting"}, {"meeting", "lunch"}}');
SELECT * FROM sal_emp;
INSERT INTO sal_emp
    VALUES ('Bill',
    ARRAY[10000, 10000, 10000, 10000],
    ARRAY[['meeting', 'lunch'], ['training', 'presentation']]);

INSERT INTO sal_emp
    VALUES ('Carol',
    ARRAY[20000, 25000, 25000, 25000],
    ARRAY[['breakfast', 'consulting'], ['meeting', 'lunch']]);
```

#### [Accessing Arrays](https://www.postgresql.org/docs/current/arrays.html#ARRAYS-ACCESSING)

```
SELECT name FROM sal_emp WHERE pay_by_quarter[1] <> pay_by_quarter[2];
SELECT pay_by_quarter[3] FROM sal_emp;
SELECT schedule[1:2][1:1] FROM sal_emp WHERE name = 'Bill';
SELECT schedule[1:2][2] FROM sal_emp WHERE name = 'Bill';
SELECT schedule[:2][2:] FROM sal_emp WHERE name = 'Bill';
SELECT schedule[:][1:1] FROM sal_emp WHERE name = 'Bill';
SELECT array_dims(schedule) FROM sal_emp WHERE name = 'Carol';
SELECT array_upper(schedule, 1) FROM sal_emp WHERE name = 'Carol';
SELECT array_length(schedule, 1) FROM sal_emp WHERE name = 'Carol';
SELECT cardinality(schedule) FROM sal_emp WHERE name = 'Carol';
```

#### [Modifying Arrays](https://www.postgresql.org/docs/current/arrays.html#ARRAYS-MODIFYING)

```
UPDATE sal_emp SET pay_by_quarter = '{25000,25000,27000,27000}'
    WHERE name = 'Carol';
UPDATE sal_emp SET pay_by_quarter = ARRAY[25000,25000,27000,27000]
    WHERE name = 'Carol';
UPDATE sal_emp SET pay_by_quarter[4] = 15000
    WHERE name = 'Bill';
UPDATE sal_emp SET pay_by_quarter[1:2] = '{27000,27000}'
    WHERE name = 'Carol';
SELECT ARRAY[1,2] || ARRAY[3,4];
SELECT ARRAY[5,6] || ARRAY[[1,2],[3,4]];
SELECT array_dims(1 || '[0:1]={2,3}'::int[]);
SELECT array_dims(ARRAY[1,2] || 3);
SELECT array_dims(ARRAY[1,2] || ARRAY[3,4,5]);
SELECT array_dims(ARRAY[[1,2],[3,4]] || ARRAY[[5,6],[7,8],[9,0]]);
SELECT array_dims(ARRAY[1,2] || ARRAY[[3,4],[5,6]]);
SELECT array_prepend(1, ARRAY[2,3]);
SELECT array_append(ARRAY[1,2], 3);
SELECT array_cat(ARRAY[1,2], ARRAY[3,4]);
SELECT array_cat(ARRAY[[1,2],[3,4]], ARRAY[5,6]);
SELECT array_cat(ARRAY[5,6], ARRAY[[1,2],[3,4]]);
SELECT ARRAY[1, 2] || '{3, 4}';  -- the untyped literal is taken as an array
SELECT ARRAY[1, 2] || NULL;                -- so is an undecorated NULL
SELECT array_append(ARRAY[1, 2], NULL);    -- this might have been meant
```

#### [Searching in Arrays](https://www.postgresql.org/docs/current/arrays.html#ARRAYS-SEARCHING)

```
SELECT * FROM sal_emp WHERE pay_by_quarter[1] = 10000 OR
                            pay_by_quarter[2] = 10000 OR
                            pay_by_quarter[3] = 10000 OR
                            pay_by_quarter[4] = 10000;
SELECT * FROM sal_emp WHERE 10000 = ANY (pay_by_quarter);
SELECT * FROM sal_emp WHERE 10000 = ALL (pay_by_quarter);
SELECT * FROM
   (SELECT pay_by_quarter,
           generate_subscripts(pay_by_quarter, 1) AS s
      FROM sal_emp) AS foo
 WHERE pay_by_quarter[s] = 10000;
SELECT * FROM sal_emp WHERE pay_by_quarter && ARRAY[10000];
SELECT array_position(ARRAY['sun','mon','tue','wed','thu','fri','sat'], 'mon');
SELECT array_positions(ARRAY[1, 4, 3, 1, 3, 4, 2, 1], 1);
```

#### [Array Input and Output Syntax](https://www.postgresql.org/docs/current/arrays.html#ARRAYS-IO)

```
SELECT f1[1][-2][3] AS e1, f1[1][-1][5] AS e2
 FROM (SELECT '[1:1][-2:-1][3:5]={{{1,2,3},{4,5,6}}}'::int[] AS f1) AS ss;
```

### [Composite Types](https://www.postgresql.org/docs/current/rowtypes.html)

#### [Declaration of Composite Types](https://www.postgresql.org/docs/current/rowtypes.html#ROWTYPES-DECLARING)

```
CREATE TYPE complex AS (
    r       double precision,
    i       double precision
);

CREATE TYPE inventory_item AS (
    name            text,
    supplier_id     integer,
    price           numeric
);

CREATE TABLE on_hand (
    item      inventory_item,
    count     integer
);

INSERT INTO on_hand VALUES (ROW('fuzzy dice', 42, 1.99), 1000);

CREATE FUNCTION price_extension(inventory_item, integer) RETURNS numeric
AS 'SELECT $1.price * $2' LANGUAGE SQL;

SELECT price_extension(item, 10) FROM on_hand;

CREATE TABLE inventory_item (
    name            text,
    supplier_id     integer REFERENCES suppliers,
    price           numeric CHECK (price > 0)
);
```

#### [Constructing Composite Values](https://www.postgresql.org/docs/current/rowtypes.html#id-1.5.7.24.6)

```
'("fuzzy dice",42,1.99)'
'("fuzzy dice",42,)'
'("",42,)'
ROW('fuzzy dice', 42, 1.99)
ROW('', 42, NULL)
('fuzzy dice', 42, 1.99)
('', 42, NULL)
```

#### [Accessing Composite Types](https://www.postgresql.org/docs/current/rowtypes.html#ROWTYPES-ACCESSING)

```
SELECT item.name FROM on_hand WHERE item.price > 9.99;
SELECT (item).name FROM on_hand WHERE (item).price > 9.99;
SELECT (on_hand.item).name FROM on_hand WHERE (on_hand.item).price > 9.99;
SELECT (my_func(...)).field FROM ...
```

#### [Modifying Composite Types](https://www.postgresql.org/docs/current/rowtypes.html#id-1.5.7.24.8)

```
INSERT INTO mytab (complex_col) VALUES((1.1,2.2));
UPDATE mytab SET complex_col = ROW(1.1,2.2) WHERE ...;
UPDATE mytab SET complex_col.r = (complex_col).r + 1 WHERE ...;
INSERT INTO mytab (complex_col.r, complex_col.i) VALUES(1.1, 2.2);
```

#### [Using Composite Types in Queries](https://www.postgresql.org/docs/current/rowtypes.html#ROWTYPES-USAGE)

```
SELECT c FROM inventory_item c;
SELECT c.* FROM inventory_item c;
SELECT c.name, c.supplier_id, c.price FROM inventory_item c;
SELECT (myfunc(x)).* FROM some_table;
SELECT (myfunc(x)).a, (myfunc(x)).b, (myfunc(x)).c FROM some_table;
SELECT somefunc(c.*) FROM inventory_item c;
SELECT somefunc(c) FROM inventory_item c;
SELECT * FROM inventory_item c ORDER BY c;
SELECT * FROM inventory_item c ORDER BY c.*;
SELECT * FROM inventory_item c ORDER BY ROW(c.*);
SELECT * FROM inventory_item c ORDER BY ROW(c.name, c.supplier_id, c.price);
SELECT * FROM inventory_item c ORDER BY (c.name, c.supplier_id, c.price);
SELECT c.name FROM inventory_item c WHERE c.price > 1000;
SELECT name(c) FROM inventory_item c WHERE price(c) > 1000;
SELECT somefunc(c) FROM inventory_item c;
SELECT somefunc(c.*) FROM inventory_item c;
SELECT c.somefunc FROM inventory_item c;
```

#### [Composite Type Input and Output Syntax](https://www.postgresql.org/docs/current/rowtypes.html#ROWTYPES-IO-SYNTAX)

```
'(  42)'
```

the whitespace will be ignored if the field type is integer, but not if it is text.

## [Functions and Operators](https://www.postgresql.org/docs/current/functions.html)

### [Sequence Manipulation Functions](https://www.postgresql.org/docs/current/functions-sequence.html)

```
SELECT currval('public.id_seq'::regclass);
SELECT nextval('public.id_seq'::regclass);
```

### [System Information Functions and Operators](https://www.postgresql.org/docs/current/functions-info.html)

```
SELECT current_database();
SELECT pg_backend_pid();
SELECT pg_blocking_pids(1);
SELECT txid_current();
```

### [System Administration Functions](https://www.postgresql.org/docs/current/functions-admin.html)

#### [Configuration Settings Functions](https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADMIN-SET)

```
SELECT current_setting('enable_nestloop');
SELECT current_setting('jit');
```

#### [Server Signaling Functions](https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADMIN-SIGNAL)

```
SELECT pg_cancel_backend(11);
SELECT pg_reload_conf();
SELECT pg_terminate_backend(11);
```

#### [Backup Control Functions](https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADMIN-BACKUP)

```
SELECT pg_switch_wal();
SELECT pg_current_wal_lsn();
SELECT pg_current_wal_insert_lsn();
SELECT pg_current_wal_flush_lsn();
SELECT pg_size_pretty(pg_wal_lsn_diff('0/16596F0','0/1659580'));
SELECT pg_walfile_name(pg_current_wal_lsn());
SELECT pg_walfile_name_offset(pg_current_wal_lsn());
SELECT pg_current_wal_lsn(),pg_current_wal_insert_lsn(),pg_current_wal_flush_lsn();
SELECT pg_current_wal_lsn(),pg_current_wal_insert_lsn(),pg_current_wal_flush_lsn(),pg_walfile_name(pg_current_wal_lsn()),pg_walfile_name_offset(pg_current_wal_lsn());

```

[9.x](https://www.postgresql.org/docs/9.6/functions-admin.html)

```
SELECT pg_switch_xlog();
SELECT pg_current_xlog_location();
SELECT pg_switch_xlog();
SELECT pg_size_pretty(pg_xlog_location_diff('0/16596F0','0/1659580'));
```

#### [Database Object Management Functions](https://www.postgresql.org/docs/12/functions-admin.html#FUNCTIONS-ADMIN-DBOBJECT)

```
SELECT * FROM pg_relation_filepath('test');
```

#### [Generic File Access Functions](https://www.postgresql.org/docs/12/functions-admin.html#FUNCTIONS-ADMIN-GENFILE)

```
SELECT * FROM pg_stat_file(pg_relation_filepath('test'));
SELECT * FROM pg_stat_file(pg_relation_filepath('test')||'_vm');
```

## [Performance Tips](https://www.postgresql.org/docs/current/performance-tips.html)

### [Populating a Database](https://www.postgresql.org/docs/current/populate.html)

#### [Disable Autocommit](https://www.postgresql.org/docs/current/populate.html#DISABLE-AUTOCOMMIT)

```
BEGIN;
INSERT...;
INSERT...;
...
COMMIT;
```

#### [Use `COPY`](https://www.postgresql.org/docs/current/populate.html#POPULATE-COPY-FROM)

[COPY](https://www.postgresql.org/docs/current/sql-copy.html)

[PREPARE](https://www.postgresql.org/docs/current/sql-prepare.html)

`COPY` is fastest when used within the same transaction as an earlier `CREATE TABLE` or `TRUNCATE` command. However, this consideration only applies when [wal_level](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-WAL-LEVEL) is `minimal` for non-partitioned tables as all commands must write WAL otherwise.

#### [Remove Indexes](https://www.postgresql.org/docs/current/populate.html#POPULATE-RM-INDEXES)

If you are loading a freshly created table, the fastest method is to create the table, bulk load the table's data using `COPY`, then create any indexes needed for the table. Creating an index on pre-existing data is quicker than updating it incrementally as each row is loaded.

#### [Remove Foreign Key Constraints](https://www.postgresql.org/docs/current/populate.html#POPULATE-RM-FKEYS)

Just as with indexes, a foreign key constraint can be checked “in bulk” more efficiently than row-by-row. 

#### [Increase `maintenance_work_mem`](https://www.postgresql.org/docs/current/populate.html#POPULATE-WORK-MEM)

This will help to speed up `CREATE INDEX` commands and `ALTER TABLE ADD FOREIGN KEY` commands. 

#### [Increase `max_wal_size`](https://www.postgresql.org/docs/current/populate.html#POPULATE-MAX-WAL-SIZE)

```
#max_wal_size = 1GB
```

#### [Disable WAL Archival and Streaming Replication](https://www.postgresql.org/docs/current/populate.html#POPULATE-PITR)

Changing these settings requires a server restart.

```
wal_level = minimal
archive_mode = off
max_wal_senders = 0
```



#### [Run `ANALYZE` Afterwards](https://www.postgresql.org/docs/current/populate.html#POPULATE-ANALYZE)

```
ANALYZE test;
VACUUM (ANALYZE) test;
```

#### [Some Notes about pg_dump](https://www.postgresql.org/docs/current/populate.html#POPULATE-PG-DUMP)

By default, pg_dump uses `COPY`.

# [Server Administration](https://www.postgresql.org/docs/current/admin.html)

## [Server Configuration](https://www.postgresql.org/docs/current/runtime-config.html)

### [Connections and Authentication](https://www.postgresql.org/docs/current/runtime-config-connection.html)

#### [Connection Settings](https://www.postgresql.org/docs/current/runtime-config-connection.html#RUNTIME-CONFIG-CONNECTION-SETTINGS)

##### max_connections

```
max_connections = 100                   # (change requires restart)
```

### [Resource Consumption](https://www.postgresql.org/docs/current/runtime-config-resource.html)

#### [Memory](https://www.postgresql.org/docs/current/runtime-config-resource.html#RUNTIME-CONFIG-RESOURCE-MEMORY)

##### shared_buffers

Sets the amount of memory the database server uses for shared memory buffers. The default is typically 128 megabytes (`128MB`), but might be less if your **kernel settings** will not support it (as determined during initdb). 

If you have a dedicated database server with 1GB or more of RAM, a reasonable starting value for `shared_buffers` is **25%** of the memory in your system.

 Larger settings for `shared_buffers` usually require a corresponding increase in `max_wal_size`, in order to spread out the process of writing large quantities of new or changed data over a longer period of time.

```
shared_buffers = 128MB                  # min 128kB
```

##### work_mem

Sets the maximum amount of memory to be used by a query operation (such as a sort or hash table) before writing to temporary disk files. Sort operations are used for `ORDER BY`, `DISTINCT`, and merge joins. Hash tables are used in hash joins, hash-based aggregation, and hash-based processing of `IN` subqueries.

```
#work_mem = 4MB                         # min 64kB
```

##### maintenance_work_mem

Specifies the maximum amount of memory to be used by maintenance operations, such as `VACUUM`, `CREATE INDEX`, and `ALTER TABLE ADD FOREIGN KEY`. 

Note that when autovacuum runs, up to [autovacuum_max_workers](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html#GUC-AUTOVACUUM-MAX-WORKERS) times this memory may be allocated, so be careful not to set the default value too high. It may be useful to control for this by separately setting [autovacuum_work_mem](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-AUTOVACUUM-WORK-MEM).

```
#maintenance_work_mem = 64MB            # min 1MB
```

##### autovacuum_work_mem

```
#autovacuum_work_mem = -1               # min 1MB, or -1 to use maintenance_work_mem
```

#### [Asynchronous Behavior](https://www.postgresql.org/docs/current/runtime-config-resource.html#RUNTIME-CONFIG-RESOURCE-ASYNC-BEHAVIOR)

##### effective_io_concurrency

Sets the number of concurrent disk I/O operations that PostgreSQL expects can be executed simultaneously. **Currently, this setting only affects bitmap heap scans.**

```
#effective_io_concurrency = 1           # 1-1000; 0 disables prefetching
```

##### max_worker_processes

Sets the maximum number of background processes that the system can support. 

```
#max_worker_processes = 8               # (change requires restart)
```

##### max_parallel_workers_per_gather

Sets the maximum number of workers that can be started by a single `Gather` or `Gather Merge` node. 

```
#max_parallel_workers_per_gather = 2    # taken from max_parallel_workers
```

##### max_parallel_maintenance_workers

Sets the maximum number of parallel workers that can be started by a single utility command. **Currently, the only parallel utility command that supports the use of parallel workers is `CREATE INDEX`, and only when building a B-tree index**. 

```
#max_parallel_maintenance_workers = 2   # taken from max_parallel_workers
```

##### max_parallel_workers

Sets the maximum number of workers that the system can support for parallel operations. 

```
#max_parallel_workers = 8               # maximum number of max_worker_processes that
                                        # can be used in parallel operations
```

### [Write Ahead Log](https://www.postgresql.org/docs/current/runtime-config-wal.html)

#### [Settings](https://www.postgresql.org/docs/current/runtime-config-wal.html#RUNTIME-CONFIG-WAL-SETTINGS)

##### wal_level

`wal_level` determines how much information is written to the WAL. 

In `minimal` level, WAL-logging of some bulk operations can be safely skipped, which can make those operations much faster (see [Section 14.4.7](https://www.postgresql.org/docs/current/populate.html#POPULATE-PITR)). Operations in which this optimization can be applied include:

| `CREATE TABLE AS`                                            |
| ------------------------------------------------------------ |
| `CREATE INDEX`                                               |
| `CLUSTER`                                                    |
| `COPY` into tables that were created or truncated in the same transaction |

```
#wal_level = replica                    # minimal, replica, or logical
                                        # (change requires restart)
```

##### fsync

If this parameter is on, the PostgreSQL server will try to make sure that updates are physically written to disk, by issuing `fsync()` system calls or various equivalent methods (see [wal_sync_method](https://www.postgresql.org/docs/current/runtime-config-wal.html#GUC-WAL-SYNC-METHOD)). This ensures that the database cluster can recover to a consistent state after an operating system or hardware crash.

```
#fsync = on                             # flush data to disk for crash safety
                                        # (turning this off can cause
                                        # unrecoverable data corruption)
```

##### synchronous_commit

Specifies whether transaction commit will wait for WAL records to be written to disk before the command returns a “success” indication to the client. Valid values are `on`, `remote_apply`, `remote_write`, `local`, and `off`. 

```
#synchronous_commit = on                # synchronization level;
                                        # off, local, remote_write, remote_apply, or on
```

This parameter can be changed at any time; the behavior for any one transaction is determined by the setting in effect when it commits. It is therefore possible, and useful, to have some transactions commit synchronously and others asynchronously. For example, to make a single multistatement transaction commit asynchronously when the default is the opposite, issue `SET LOCAL synchronous_commit TO OFF` within the transaction.

```
postgres=# SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE name='synchronous_commit';
-[ RECORD 1 ]---+---------------------------
name            | synchronous_commit
setting         | on
unit            | 
context         | user
boot_val        | on
reset_val       | on
source          | default
pending_restart | f
category        | Write-Ahead Log / Settings
```

##### wal_buffers

The amount of shared memory used for WAL data that has not yet been written to disk. The default setting of -1 selects a size equal to 1/32nd (about 3%) of [shared_buffers](https://www.postgresql.org/docs/current/runtime-config-resource.html#GUC-SHARED-BUFFERS), but not less than `64kB` nor more than the size of one WAL segment, typically `16MB`. 

```
#wal_buffers = -1                       # min 32kB, -1 sets based on shared_buffers
                                        # (change requires restart)
```

##### wal_writer_delay

Specifies how often the WAL writer flushes WAL, in time terms. After flushing WAL the writer sleeps for the length of time given by `wal_writer_delay`. 

```
#wal_writer_delay = 200ms               # 1-10000 milliseconds
```

#### [Checkpoints](https://www.postgresql.org/docs/current/runtime-config-wal.html#RUNTIME-CONFIG-WAL-CHECKPOINTS)

##### checkpoint_timeout

Maximum time between automatic WAL checkpoints.

```
#checkpoint_timeout = 5min              # range 30s-1d
```

##### checkpoint_completion_target

Specifies the target of checkpoint completion, as a fraction of total time between checkpoints. 

```
#checkpoint_completion_target = 0.5     # checkpoint target duration, 0.0 - 1.0
```

##### max_wal_size

Maximum size to let the WAL grow to between automatic WAL checkpoints. This is a soft limit; WAL size can exceed `max_wal_size` under special circumstances, such as heavy load, a failing `archive_command`, or a high `wal_keep_segments` setting. If this value is specified without units, it is taken as megabytes. 

```
#max_wal_size = 1GB
```

##### min_wal_size

As long as WAL disk usage stays below this setting, old WAL files are always recycled for future use at a checkpoint, rather than removed. This can be used to ensure that enough WAL space is reserved to handle spikes in WAL usage, for example when running large batch jobs.

```
#min_wal_size = 80MB
```

#### [Archiving](https://www.postgresql.org/docs/current/runtime-config-wal.html#RUNTIME-CONFIG-WAL-ARCHIVING)

```
#archive_mode = off		# enables archiving; off, on, or always
				# (change requires restart)
#archive_command = ''		# command to use to archive a logfile segment
				# placeholders: %p = path of file to archive
				#               %f = file name only
				# e.g. 'test ! -f /mnt/server/archivedir/%f && cp %p /mnt/server/archivedir/%f'
#archive_timeout = 0		# force a logfile segment switch after this
				# number of seconds; 0 disables
```

### [Replication](https://www.postgresql.org/docs/current/runtime-config-replication.html)

### [Query Planning](https://www.postgresql.org/docs/current/runtime-config-query.html)

#### [Planner Method Configuration](https://www.postgresql.org/docs/current/runtime-config-query.html#RUNTIME-CONFIG-QUERY-ENABLE)

Better ways to improve the quality of the plans chosen by the optimizer include adjusting the planner cost constants (see [Planner Cost Constants](https://www.postgresql.org/docs/current/runtime-config-query.html#RUNTIME-CONFIG-QUERY-CONSTANTS)), running [ANALYZE](https://www.postgresql.org/docs/current/sql-analyze.html) manually, increasing the value of the [default_statistics_target](https://www.postgresql.org/docs/current/runtime-config-query.html#GUC-DEFAULT-STATISTICS-TARGET) configuration parameter, and increasing the amount of statistics collected for specific columns using `ALTER TABLE SET STATISTICS`.

```
#enable_hashjoin = on
#enable_mergejoin = on
#enable_nestloop = on
```

#### [Planner Cost Constants](https://www.postgresql.org/docs/current/runtime-config-query.html#RUNTIME-CONFIG-QUERY-CONSTANTS)

##### seq_page_cost

Sets the planner's estimate of the cost of a disk page fetch that is part of a series of sequential fetches.

```
#seq_page_cost = 1.0                    # measured on an arbitrary scale
```

##### random_page_cost

Sets the planner's estimate of the cost of a non-sequentially-fetched disk page. 

The default value can be thought of as modeling random access as 40 times slower than sequential, while expecting 90% of random reads to be cached.

```
#random_page_cost = 4.0                 # same scale as above
```

##### min_parallel_table_scan_size

Sets the minimum amount of table data that must be scanned in order for a parallel scan to be considered. 

```
#min_parallel_table_scan_size = 8MB
```

##### min_parallel_index_scan_size

Sets the minimum amount of index data that must be scanned in order for a parallel scan to be considered.

```
#min_parallel_index_scan_size = 512kB
```

##### effective_cache_size

Sets the planner's assumption about the effective size of the disk cache that is available to a single query. This is factored into estimates of the cost of using an index; a higher value makes it more likely **index scans** will be used, a lower value makes it more likely **sequential scans** will be used. **This parameter has no effect on the size of shared memory allocated by PostgreSQL, nor does it reserve kernel disk cache; it is used only for estimation purposes.**

```
#effective_cache_size = 4GB
```

#### [Other Planner Options](https://www.postgresql.org/docs/current/runtime-config-query.html#RUNTIME-CONFIG-QUERY-OTHER)

Sets the default statistics target for table columns without a column-specific target set via `ALTER TABLE SET STATISTICS`. Larger values increase the time needed to do `ANALYZE`, but might improve the quality of the planner's estimates. The default is 100. For more information on the use of statistics by the PostgreSQL query planner, refer to [Statistics Used by the Planner](https://www.postgresql.org/docs/current/planner-stats.html).

```
#default_statistics_target = 100        # range 1-10000
```

### [Automatic Vacuuming](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html)

```
#autovacuum = on			# Enable autovacuum subprocess?  'on'
					# requires track_counts to also be on.
#log_autovacuum_min_duration = -1	# -1 disables, 0 logs all actions and
					# their durations, > 0 logs only
					# actions running at least this number
					# of milliseconds.
#autovacuum_max_workers = 3		# max number of autovacuum subprocesses
					# (change requires restart)
#autovacuum_naptime = 1min		# time between autovacuum runs
#autovacuum_vacuum_threshold = 50	# min number of row updates before
					# vacuum
#autovacuum_analyze_threshold = 50	# min number of row updates before
					# analyze
#autovacuum_vacuum_scale_factor = 0.2	# fraction of table size before vacuum
#autovacuum_analyze_scale_factor = 0.1	# fraction of table size before analyze
#autovacuum_freeze_max_age = 200000000	# maximum XID age before forced vacuum
					# (change requires restart)

```

## [High Availability, Load Balancing, and Replication](https://www.postgresql.org/docs/current/high-availability.html)

### [Log-Shipping Standby Servers](https://www.postgresql.org/docs/current/warm-standby.html)



## [Monitoring Database Activity](https://www.postgresql.org/docs/current/monitoring.html)

### [Standard Unix Tools](https://www.postgresql.org/docs/current/monitoring-ps.html)

```
ps -ef|grep -v grep|grep postgres:
top
iostat
vmstat
netstat -tpc
```

### [The Statistics Collector](https://www.postgresql.org/docs/current/monitoring-stats.html)

#### [Statistics Collection Configuration](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-STATS-SETUP)

```
# - Query and Index Statistics Collector -

#track_activities = on
#track_counts = on
#track_io_timing = off
#track_functions = none                 # none, pl, all
#track_activity_query_size = 1024       # (change requires restart)
stats_temp_directory = '/var/run/postgresql/12-main.pg_stat_tmp'
```

#### [Viewing Statistics](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-STATS-VIEWS)

##### Dynamic Statistics Views

###### pg_stat_activity

```
SELECT * FROM pg_stat_activity
WHERE pid <> pg_backend_pid()
AND backend_type like '%client%';

SELECT * FROM pg_stat_activity
WHERE pid = ANY(pg_blocking_pids(17276));

SELECT * FROM pg_stat_activity
WHERE state <> 'idle'
AND query IS NOT NULL
AND pid <> pg_backend_pid();
SELECT * FROM pg_stat_activity
WHERE usename = 'user1';
```

```
SELECT
    *
FROM
    pg_stat_activity
WHERE
    usename = 'user1'
    AND state <> 'idle';
```

```
--group by client
SELECT
    datname,
    usename,
    application_name,
    client_addr,
    client_hostname,
    state,
    COUNT(1)
FROM
    pg_stat_activity
WHERE
    state <> 'idle'
    AND query NOT LIKE '% FROM pg_stat_activity %'
GROUP BY
    datname,
    usename,
    application_name,
    client_addr,
    client_hostname,
    state
ORDER BY
    datname,
    usename,
    application_name,
    client_addr,
    client_hostname,
    state;
```

```
SELECT * FROM pg_stat_replication;
```

##### Collected Statistics Views

###### pg_stat_all_tables

```
SELECT * FROM pg_stat_all_tables WHERE relname='test';
SELECT pg_stat_get_dead_tuples(oid) FROM pg_class WHERE relname='test';
```

# [Server Programming](https://www.postgresql.org/docs/current/server-programming.html)

## [PL/pgSQL - SQL Procedural Language](https://www.postgresql.org/docs/current/plpgsql.html)

### [Cursors](https://www.postgresql.org/docs/current/plpgsql-cursors.html)

#### [Using Cursors](https://www.postgresql.org/docs/current/plpgsql-cursors.html#PLPGSQL-CURSOR-USING)

##### Returning Cursors

```
CREATE TABLE test (col text);
INSERT INTO test VALUES ('123');

CREATE FUNCTION reffunc(refcursor) RETURNS refcursor AS '
BEGIN
    OPEN $1 FOR SELECT col FROM test;
    RETURN $1;
END;
' LANGUAGE plpgsql;

BEGIN;
SELECT reffunc('funccursor');
FETCH ALL IN funccursor;
COMMIT;
```

**Procedure** with INOUT refcursor

```
CREATE TABLE test (col text);
INSERT INTO test VALUES ('123');

CREATE PROCEDURE myproc(ref_cur INOUT refcursor) AS
$$
BEGIN
    OPEN ref_cur FOR SELECT * FROM test;
END;
$$ LANGUAGE plpgsql;

DO
$$
DECLARE
my_cur refcursor;
c1 text;
BEGIN
CALL myproc(my_cur);
FETCH my_cur INTO c1;
LOOP
EXIT WHEN NOT FOUND;
RAISE NOTICE 'c1:%',c1;
FETCH my_cur INTO c1;
END LOOP;
CLOSE my_cur;
END;
$$;
```

The following example uses automatic cursor name generation:

```
CREATE FUNCTION reffunc2() RETURNS refcursor AS '
DECLARE
    ref_cur refcursor;
BEGIN
    OPEN ref_cur FOR SELECT col FROM test;
    RETURN ref_cur;
END;
' LANGUAGE plpgsql;

-- need to be in a transaction to use cursors.
BEGIN;
SELECT reffunc2();

      reffunc2
--------------------
 <unnamed cursor 1>
(1 row)

FETCH ALL IN "<unnamed portal 1>";
COMMIT;
```

The following example shows one way to return multiple cursors from a single function:

```
CREATE FUNCTION myfunc(refcursor, refcursor) RETURNS SETOF refcursor AS $$
BEGIN
    OPEN $1 FOR SELECT * FROM table_1;
    RETURN NEXT $1;
    OPEN $2 FOR SELECT * FROM table_2;
    RETURN NEXT $2;
END;
$$ LANGUAGE plpgsql;

-- need to be in a transaction to use cursors.
BEGIN;

SELECT * FROM myfunc('a', 'b');

FETCH ALL FROM a;
FETCH ALL FROM b;
COMMIT;
```

Open cursors in a different way for calling by Java, etc.

```
CREATE FUNCTION myfunc2(INOUT refcursor, INOUT refcursor) RETURNS RECORD AS $$
BEGIN
    OPEN $1 FOR SELECT * FROM table_1;
    OPEN $2 FOR SELECT * FROM table_2;
END;
$$ LANGUAGE plpgsql;

-- need to be in a transaction to use cursors.
BEGIN;

SELECT * FROM myfunc('a', 'b');

FETCH ALL FROM a;
FETCH ALL FROM b;
COMMIT;
```

### [Tips for Developing in PL/pgSQL](https://www.postgresql.org/docs/current/plpgsql-development-tips.html)

#### [Handling of Quotation Marks](https://www.postgresql.org/docs/current/plpgsql-development-tips.html#PLPGSQL-QUOTE-TIPS)

```
CREATE OR REPLACE FUNCTION testfunc(integer) RETURNS integer AS $PROC$
          ....
$PROC$ LANGUAGE plpgsql;

CREATE FUNCTION foo() RETURNS integer AS '
          ....
' LANGUAGE plpgsql;

a_output := a_output || '' AND name LIKE ''''foobar'''' AND xyz''
a_output := a_output || $$ AND name LIKE 'foobar' AND xyz$$
```

#### [Additional Compile-Time and Run-Time Checks](https://www.postgresql.org/docs/current/plpgsql-development-tips.html#PLPGSQL-EXTRA-CHECKS)

```
SET plpgsql.extra_warnings TO 'all';
SET plpgsql.extra_errors TO 'all';
SET plpgsql.extra_warnings TO 'shadowed_variables';
SET plpgsql.extra_warnings TO 'strict_multi_assignment';
SET plpgsql.extra_warnings TO 'too_many_rows';

CREATE FUNCTION foo(f1 int) RETURNS int AS $$
DECLARE
f1 int;
BEGIN
RETURN f1;
END
$$ LANGUAGE plpgsql;
WARNING:  variable "f1" shadows a previously defined variable
LINE 3: f1 int;
        ^
CREATE FUNCTION
```

```
SET plpgsql.extra_warnings TO 'strict_multi_assignment';

CREATE OR REPLACE FUNCTION public.foo()
 RETURNS void
 LANGUAGE plpgsql
AS $$
DECLARE
  x int;
  y int;
BEGIN
  SELECT 1 INTO x, y;
  SELECT 1, 2 INTO x, y;
  SELECT 1, 2, 3 INTO x, y;
END;
$$;

SELECT foo();
WARNING:  number of source and target fields in assignment does not match
DETAIL:  strict_multi_assignment check of extra_warnings is active.
HINT:  Make sure the query returns the exact list of columns.
WARNING:  number of source and target fields in assignment does not match
DETAIL:  strict_multi_assignment check of extra_warnings is active.
HINT:  Make sure the query returns the exact list of columns.

 foo 
-----
 
(1 row)
```

### [Porting from Oracle PL/SQL](https://www.postgresql.org/docs/current/plpgsql-porting.html)

# [Reference](https://www.postgresql.org/docs/current/reference.html)

## [SQL Commands](https://www.postgresql.org/docs/current/sql-commands.html)

### [ALTER SEQUENCE](https://www.postgresql.org/docs/current/sql-altersequence.html)

#### OWNER

```
ALTER SEQUENCE public.id_seq OWNED BY public.test.id;
```

### [ALTER SYSTEM](https://www.postgresql.org/docs/12/sql-altersystem.html)

```
ALTER SYSTEM SET default_transaction_read_only=on;
ALTER SYSTEM RESET default_transaction_read_only;
```

```
ALTER SYSTEM writes the given parameter setting to the postgresql.auto.conf file, which is read in addition to postgresql.conf.
```

### [ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html)

#### OWNER

```
ALTER TABLE public.test OWNER TO aa_owners;
```

#### DEFAULT

```
ALTER TABLE ONLY public.test ALTER COLUMN id SET DEFAULT nextval('public.id_seq'::regclass);
```

#### COLUMN

```
ALTER TABLE public.test ADD COLUMN c1 BOOLEAN;
```

#### CONSTRAINT

```
ALTER TABLE ONLY public.test ADD CONSTRAINT ck_test PRIMARY KEY (id);
```

#### RENAME

```
ALTER TABLE public.test RENAME TO test2;
```

### [BEGIN](https://www.postgresql.org/docs/current/sql-begin.html)

```
BEGIN;
...
COMMIT;
```

### **[CHECKPOINT](https://www.postgresql.org/docs/current/sql-checkpoint.html)**

```
CHECKPOINT;
```

### **[COPY](https://www.postgresql.org/docs/current/sql-copy.html)**

```
COPY country FROM '/usr1/proj/bray/sql/country_data';
```

### **[CREATE INDEX](https://www.postgresql.org/docs/current/sql-createindex.html)**

```
CREATE INDEX idx_test ON public.test USING btree (id, name);
CREATE INDEX CONCURRENTLY idx_test ON public.test USING btree (id, name);
CREATE UNIQUE INDEX CONCURRENTLY uk_test ON public.test USING btree (name);
CREATE INDEX CONCURRENTLY idx_test ON public.test USING btree (lower((name)::text) varchar_pattern_ops);
CREATE INDEX CONCURRENTLY idx_test ON public.test USING gin (to_tsvector('gender'::regconfig, (name)::text));
```

### [CREATE TYPE](https://www.postgresql.org/docs/current/sql-createtype.html)

```
CREATE TYPE compfoo AS (f1 int, f2 text);

CREATE FUNCTION getfoo() RETURNS SETOF compfoo AS $$
    SELECT fooid, fooname FROM foo
$$ LANGUAGE SQL;

CREATE TYPE bug_status AS ENUM ('new', 'open', 'closed');

CREATE TABLE bug (
    id serial,
    description text,
    status bug_status
);

CREATE TYPE float8_range AS RANGE (subtype = float8, subtype_diff = float8mi);

CREATE TYPE box;

CREATE FUNCTION my_box_in_function(cstring) RETURNS box AS ... ;
CREATE FUNCTION my_box_out_function(box) RETURNS cstring AS ... ;

CREATE TYPE box (
    INTERNALLENGTH = 16,
    INPUT = my_box_in_function,
    OUTPUT = my_box_out_function
);

CREATE TABLE myboxes (
    id integer,
    description box
);

CREATE TYPE box (
    INTERNALLENGTH = 16,
    INPUT = my_box_in_function,
    OUTPUT = my_box_out_function,
    ELEMENT = float4
);

CREATE TYPE bigobj (
    INPUT = lo_filein, OUTPUT = lo_fileout,
    INTERNALLENGTH = VARIABLE
);
CREATE TABLE big_objs (
    id integer,
    obj bigobj
);
```

### **[DO](https://www.postgresql.org/docs/current/sql-do.html)**

```
DO
$$
DECLARE
ln int:= 0;
BEGIN
...
END
$$;
```

### **[GRANT](https://www.postgresql.org/docs/current/sql-grant.html)**

```
GRANT SELECT ON TABLE public.test TO alvin;
GRANT SELECT,INSERT,DELETE,UPDATE ON TABLE public.test TO alvin;
```

### [LOCK](https://www.postgresql.org/docs/current/sql-lock.html)

```
LOCK TABLE public.test IN SHARE MODE;
LOCK TABLE public.test IN ACCESS EXCLUSIVE MODE;
```

### [PREPARE](https://www.postgresql.org/docs/current/sql-prepare.html)

```
PREPARE fooplan (int, text, bool, numeric) AS
    INSERT INTO foo VALUES($1, $2, $3, $4);
EXECUTE fooplan(1, 'Hunter Valley', 't', 200.00);
```

### [REINDEX](https://www.postgresql.org/docs/current/sql-reindex.html)

```
REINDEX ( VERBOSE ) INDEX CONCURRENTLY idx_test_a1;
REINDEX ( VERBOSE ) TABLE CONCURRENTLY public.test;
```

### [REVOKE](https://www.postgresql.org/docs/current/sql-revoke.html)

```
REVOKE ALL ON TABLE public.test FROM PUBLIC;
REVOKE ALL ON TABLE public.test FROM alvin;
```

### **[SET](https://www.postgresql.org/docs/current/sql-set.html)**

```
SET enable_nestloop = off;
SET SESSION enable_nestloop = off;
SET LOCAL enable_nestloop = off;
```

### [VACUUM](https://www.postgresql.org/docs/current/sql-vacuum.html)

```
VACUUM (VERBOSE, ANALYZE) test;
VACUUM (VERBOSE, FREEZE) test;
VACUUM (VERBOSE, SKIP_LOCKED) test;
VACUUM (FREEZE) test;
```

## [PostgreSQL Client Applications](https://www.postgresql.org/docs/current/reference-client.html)

### [pg_basebackup](https://www.postgresql.org/docs/current/app-pgbasebackup.html)

```
pg_basebackup -h 10.2.10.xxx -U replicator -v -c fast -F p -P -X s -R -D /var/lib/postgresql/12/main

pg_basebackup -h 10.2.10.xxx-U replicator -v -c fast -F p -P -X s -R -D /var/lib/postgresql/12/main --xlogdir=/var/lib/pg_wal/wal/
```

In postgres 12, there are no recovery.conf but `standby.signal` and `postgresql.auto.conf`.

```
-R
--write-recovery-conf
Create standby.signal and append connection settings to postgresql.auto.conf in the output directory (or into the base archive file when using tar format) to ease setting up a standby server. The postgresql.auto.conf file will record the connection settings and, if specified, the replication slot that pg_basebackup is using, so that the streaming replication will use the same settings later on.
```

### [pgbench](https://www.postgresql.org/docs/current/pgbench.html)

By default, pgbench tests a scenario that is loosely based on [TPC-B](http://www.tpc.org/tpcb/), involving five `SELECT`, `UPDATE`, and `INSERT` commands per transaction. 

```
pgbench -i test
pgbench -i -s 20 test
pgbench -i -I dtp -d test
pgbench -r -P 1 -T 10 test
pgbench -r -P 1 -c 30 -T 10 test
pgbench -r -P 1 -c 4 -j 2 -T 10 test
pgbench -r -P 1 -c 4 -j 2 -T 10 -S test
pgbench -r -P 1 -c 32 -j 32 -T 120 -M prepared -v -f ./ro.sql -D scale=10000
pgbench -i -I d -d test
```

### [pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html)

```
pg_dump -d testdb -p 5432 -t test -f test.sql -v -s
pg_dump -d testdb -p 5432 -t test -f test.sql -v -s -x
pg_dump -d testdb -p 5432 -t test -Fc -f test.dump -v -a
```

### [pg_restore](https://www.postgresql.org/docs/current/app-pgrestore.html)

```
pg_restore -d testdb -p 5432 test.dump -v
```

### [psql](https://www.postgresql.org/docs/current/app-psql.html)

```
psql -d testdb -p 5432 -f test.sql
```

## [PostgreSQL Server Applications](https://www.postgresql.org/docs/current/reference-server.html)

```
cd /usr/lib/postgresql/12/bin
```

### [pg_archivecleanup](https://www.postgresql.org/docs/current/pgarchivecleanup.html)

```
./pg_archivecleanup -d /var/lib/postgresql/12/main/pg_wal/ 00000001000000010000008E
```

### [pg_controldata](https://www.postgresql.org/docs/current/app-pgcontroldata.html)

```
./pg_controldata /var/lib/postgresql/12/main
./pg_controldata -D /var/lib/postgresql/12/main
```

### [pg_waldump](https://www.postgresql.org/docs/current/pgwaldump.html)

```
cd /usr/lib/postgresql/12/bin
./pg_waldump ~/12/main/pg_wal/0000000100000001000000DB >/tmp/0000000100000001000000DB.dump
```

# [Internals](https://www.postgresql.org/docs/current/internals.html)

## [System Catalogs](https://www.postgresql.org/docs/current/catalogs.html)

### [`pg_class`](https://www.postgresql.org/docs/current/catalog-pg-class.html)

```
SELECT * from pg_class where relname='test';
SELECT relname,relminmxid,age(relminmxid),relfrozenxid,age(relfrozenxid) from pg_class where relname='test';
SELECT oid,relname,relfilenode,reltoastrelid from pg_class where relname='test';
```

```
age(relfrozenxid) = txid_current() - relfrozenxid
```

### [`pg_database`](https://www.postgresql.org/docs/current/catalog-pg-database.html)

```
SELECT datname, datfrozenxid,age(datfrozenxid) FROM pg_database;
```

```
age(datfrozenxid) = txid_current() - datfrozenxid
```

### [`pg_file_settings`](https://www.postgresql.org/docs/current/view-pg-file-settings.html)

```
SELECT * FROM pg_file_settings;
SELECT * FROM pg_file_settings WHERE NOT applied;
```

According to [Parameter Interaction via the Configuration File](https://www.postgresql.org/docs/12/config-setting.html#CONFIG-SETTING-CONFIGURATION-FILE),

```
The system view pg_file_settings can be helpful for pre-testing changes to the configuration files, or for diagnosing problems if a SIGHUP signal did not have the desired effects.
```

### [`pg_locks`](https://www.postgresql.org/docs/current/view-pg-locks.html)

```
SELECT pid, t.relkind, t.relname, mode, granted
FROM pg_locks l JOIN pg_class t ON l.relation = t.oid
WHERE t.relkind = 'r'
    AND t.relname = 'app';

SELECT pid, t.relkind, t.relname, mode, granted
FROM pg_locks l JOIN pg_class t ON l.relation = t.oid
WHERE pid = ANY(pg_blocking_pids(17276));
```

### [`pg_settings`](https://www.postgresql.org/docs/current/view-pg-settings.html)

```
SELECT * FROM pg_settings WHERE name='enable_nestloop';
SELECT * FROM pg_settings WHERE name='jit';
SELECT name,setting FROM pg_settings WHERE name in ('jit','enable_nestloop','work_mem','max_files_per_process');

SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE pending_restart is true;

SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE name='jit';
SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE name='min_parallel_index_scan_size';
SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE name='min_parallel_table_scan_size';
SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE name='block_size';
SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE name='max_files_per_process';

SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE name IN ('jit','enable_nestloop','work_mem','max_files_per_process');

SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE name IN ('min_parallel_table_scan_size','min_parallel_index_scan_size','work_mem');

SELECT context,count(1) FROM pg_settings GROUP BY context;
SELECT context,source,count(1) FROM pg_settings GROUP BY context,source;
SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE context='user';
SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE context='postmaster';
SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE context='postmaster' AND boot_val<>reset_val ;

SELECT name,setting,unit,context,boot_val,reset_val,category FROM pg_settings WHERE boot_val<>reset_val ORDER BY category,name;
SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE category LIKE '%Tuning%' ORDER BY category,name;
SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE category LIKE '%Resource%' ORDER BY category,name;
SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE category LIKE '%Tuning%' AND boot_val<>reset_val ORDER BY category,name;
SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE category LIKE '%Resource%' AND boot_val<>reset_val ORDER BY category,name;

SELECT pending_restart,count(1) FROM pg_settings GROUP BY pending_restart;
SELECT sourcefile,count(1) FROM pg_settings GROUP BY sourcefile;
SELECT source,sourcefile,count(1) FROM pg_settings GROUP BY source,sourcefile;
SELECT category,count(1) FROM pg_settings GROUP BY category ORDER BY count(1) DESC;

SELECT source,count(1) FROM pg_settings GROUP BY source;
SELECT name,setting,unit,context,boot_val,reset_val,source,pending_restart,category FROM pg_settings WHERE source='configuration file';

UPDATE pg_settings SET setting='on' WHERE name='jit';
UPDATE pg_settings SET setting='off' WHERE name='jit';
```

## [Database Physical Storage](https://www.postgresql.org/docs/current/storage.html)

### [TOAST](https://www.postgresql.org/docs/current/storage-toast.html)

```
SELECT * FROM pg_toast.pg_toast_16385;

\! oid2name -d postgres -f 102420
```

# [Appendixes](https://www.postgresql.org/docs/current/appendixes.html)

## [Additional Supplied Modules](https://www.postgresql.org/docs/current/contrib.html)

### [pageinspect](https://www.postgresql.org/docs/current/pageinspect.html)

```
CREATE EXTENSION pageinspect;
```

#### [Heap Functions](https://www.postgresql.org/docs/current/pageinspect.html#id-1.11.7.31.5)

```
SELECT * FROM heap_page_items(get_raw_page('test', 0));
```

#### [B-Tree Functions](https://www.postgresql.org/docs/current/pageinspect.html#id-1.11.7.31.6)

```
SELECT * FROM bt_page_items('index_name', 1);
```

### [pgstattuple](https://www.postgresql.org/docs/current/pgstattuple.html)

```
CREATE EXTENSION pgstattuple;
SELECT * FROM pgstattuple('public.test');
```

### [pg_visibility](https://www.postgresql.org/docs/current/pgvisibility.html)

```
CREATE EXTENSION pg_visibility;
SELECT * FROM pg_visibility_map('public.test', 0);
```

## [Additional Supplied Programs](https://www.postgresql.org/docs/current/contrib-prog.html)

### [Client Applications](https://www.postgresql.org/docs/current/contrib-prog-client.html)

#### **[oid2name](https://www.postgresql.org/docs/current/oid2name.html)**

```
sudo apt install postgresql-contrib
--oid2name -d test -f 102420
```

# Others

## [plpgsql_check](https://pgxn.org/dist/plpgsql_check/)

The plpgsql_check is PostgreSQL extension with functionality for direct or indirect extra validation of functions in plpgsql language. It verifies a validity of SQL identifiers used in plpgsql code. It try to identify a performance issues. Modern versions has integrated profiler. The table and function dependencies can be displayed

