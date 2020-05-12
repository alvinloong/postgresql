# [The SQL Language](https://www.postgresql.org/docs/current/sql.html)

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

# [Server Administration](https://www.postgresql.org/docs/current/admin.html)

## [Server Configuration](https://www.postgresql.org/docs/current/runtime-config.html)



### [Write Ahead Log](https://www.postgresql.org/docs/current/runtime-config-wal.html)

### [Replication](https://www.postgresql.org/docs/current/runtime-config-replication.html)

### [Automatic Vacuuming](https://www.postgresql.org/docs/current/runtime-config-autovacuum.html)

​	

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

### **[CHECKPOINT](https://www.postgresql.org/docs/current/sql-checkpoint.html)**

```
CHECKPOINT;
```

### **[CREATE INDEX](https://www.postgresql.org/docs/current/sql-createindex.html)**

```
CREATE INDEX idx_test ON public.test USING btree (id, name);
CREATE INDEX CONCURRENTLY idx_test ON public.test USING btree (id, name);
CREATE UNIQUE INDEX CONCURRENTLY uk_test ON public.test USING btree (name);
CREATE INDEX CONCURRENTLY idx_test ON public.test USING btree (lower((name)::text) varchar_pattern_ops);
CREATE INDEX CONCURRENTLY idx_test ON public.test USING gin (to_tsvector('gender'::regconfig, (name)::text));
```

### **[DO](https://www.postgresql.org/docs/current/sql-do.html)**

```
BEGIN;
ALTER TABLE public.test2 RENAME TO test2_new;
ALTER TABLE public.test2_new RENAME TO test2;
COMMIT;
```

```
DO
$$
DECLARE
nn int:= 0 ;
l_trash_var    text ;
BEGIN

LOCK TABLE test2 IN ACCESS EXCLUSIVE MODE;
ALTER TABLE public.test2 RENAME TO test17_new;
ALTER TABLE public.test16_new RENAME TO test2;

WHILE nn <= 5 LOOP
nn = nn +1 ;
SELECT pg_sleep(1) INTO l_trash_var ;
RAISE NOTICE 'sleep % seconds', nn ; 
END LOOP ;

END
$$;
```



```
DO
$$
DECLARE
nn int:= 0 ;
nn2 int:= 0 ;
l_trash_var    text ;
BEGIN
SELECT n1 INTO STRICT nn2 FROM test2 WHERE id=1;
WHILE nn <= 10    LOOP
nn = nn +1 ;
SELECT n1 INTO STRICT nn2 FROM test2 WHERE id=1;
SELECT pg_sleep(1) INTO l_trash_var ;
RAISE NOTICE 'sleep % seconds', nn ; 
END LOOP ;
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

