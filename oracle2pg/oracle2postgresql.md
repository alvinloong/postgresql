# Oracle vs PostgreSQL

| Oracle                                    | PostgreSQL                                    |
| ----------------------------------------- | --------------------------------------------- |
| user                                      | user and schema                               |
| dblink                                    | dblink/oracle_fdw/postgres_fdw                |
| EXTERNAL TABLES                           | file_fdw                                      |
| SYNONYM                                   | VIEW                                          |
| ROWNUM                                    | Need to be rewritten as window function       |
| ending ROWNUM                             | LIMIT/OFFSET                                  |
| '' IS NULL                                | '' IS NOT NULL                                |
| global variables                          | set_config/use dedicated tables               |
| package                                   | use name package_procedure or use schema      |
| anonymous/initialization block in package | use an init function with this code           |
| autonomous transaction                    | not natively supported                        |
| jobs scheduler                            | pgAgent / pg_cron                             |
| UTL_SMTP                                  | rewrite in extended language like Python/Perl |
| SQLPLUS                                   | PSQL                                          |
| TOAD / Oracle SQL Developper              | pgAdmin                                       |
| exp                                       | pg_dump/pg_dumpall                            |
| imp                                       | pg_restore or psql                            |
| RMAN                                      | Barman/wal-e                                  |
| Data Guard                                | PostgreSQL replication/PgPool/Supervisord     |

# Data Types

| Oracle                         | PostgreSQL                    |
| ------------------------------ | ----------------------------- |
| DATE                           | DATE/TIMESTAMP                |
| VARCHAR2                       | VARCHAR                       |
| INTEGER                        | INTEGER                       |
| PLS_INTEGER                    | INTEGER                       |
| SMALLINT                       | SMALLINT                      |
| NUMBER                         | BIGINT/NUMERIC(P,S)           |
| NUMBER(P,S)                    | NUMERIC(P,S)/DOUBLE PRECISION |
| DECIMAL                        | DECIMAL                       |
| CLOB                           | TEXT                          |
| RAW                            | BYTEA                         |
| FLOAT                          | DOUBLE PRECISION              |
| DOUBLE PRECISION               | DOUBLE PRECISION              |
| TIMESTAMP                      | TIMESTAMP                     |
| TIMESTAMP WITH LOCAL TIME ZONE | TIMESTAMP WITH TIME ZONE      |
| TIMESTAMP WITH TIME ZONE       | TIMESTAMP WITH TIME ZONE      |
| INTERVAL DAY (4) TO SECOND (6) | INTERVAL DAY TO SECOND(6)     |

# Syntax

## SQL

| Oracle                                                       | PostgreSQL                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| SELECT sysdate FROM dual;                                    | SELECT now(); SELECT CLOCK_TIMESTAMP();                      |
| SELECT rownum,rowid FROM dual;                               | SELECT ctid,id,name FROM company;                            |
| SELECT company_id_seq.nextval FROM dual;                     | SELECT nextval('company_id_seq');                            |
| SELECT DECODE('a','a','aa','default') dd FROM dual;--aa<br />SELECT CASE 'a' WHEN 'a' THEN 'aa' ELSE 'default' END dd FROM dual;--aa | SELECT CASE 'a' WHEN 'a' THEN 'aa' ELSE 'default' END dd; --aa |
| SELECT DECODE(NULL,NULL,'NULL','default') dd FROM dual;--NULL<br />SELECT CASE NULL WHEN NULL THEN 'NULL' ELSE 'default' END dd FROM dual;--default | SELECT CASE NULL WHEN NULL THEN 'NULL' ELSE 'default' END dd; --default |
| SELECT CASE WHEN NULL IS NULL THEN 'NULL' ELSE 'default' END dd FROM dual;--default | SELECT CASE WHEN NULL IS NULL THEN 'NULL' ELSE 'default' END dd; --NULL |
| SELECT NVL(NULL,'IS NULL') nn FROM dual;--IS NULL<br />SELECT COALESCE(NULL,'IS NULL') nn FROM dual;--IS NULL<br />SELECT NVL2(NULL,'NOT NULL','IS NULL') nn FROM dual;--IS NULL | SELECT COALESCE(NULL,'IS NULL') nn;--IS NULL                 |
| --The alias for subquery is not mandatory<br/>SELECT * FROM (SELECT * FROM dual); | --An alias for subquery must be provided<br/>SELECT * FROM (SELECT 'X') a; |
| -- Outer Joins<br/>SELECT a.field1, b.field2 FROM a, b WHERE a.item_id = b.item_id(+);<br/>SELECT a.field1, b.field2 FROM a LEFT OUTER JOIN b ON a.item_id = b.item_id; | -- Outer Joins<br/>SELECT a.field1, b.field2 FROM a LEFT OUTER JOIN b ON a.item_id = b.item_id; |
| --CONNECT BY<br/>SELECT level FROM dual CONNECT BY rownum <=10; | --RECURSIVE<br/>WITH RECURSIVE t(n) AS<br/>  ( SELECT 1<br/>  UNION<br/>  SELECT n+1 FROM t WHERE n < 10<br/>  )<br/>SELECT * FROM t; |
| --NULL vs empty string<br/>SELECT * FROM dual WHERE '' IS NULL;--X<br/>SELECT CASE WHEN '' IS NULL THEN 'NULL' ELSE 'default' END dd FROM dual;--NULL | --NULL vs empty string<br/>SELECT '' IS NULL;--f<br/>SELECT '' IS NOT NULL;--t<br/>SELECT CASE WHEN '' IS NULL THEN 'NULL' ELSE 'default' END dd;--default |
| INSERT INTO company (id,name,age,address) VALUES (3, 'Allen3', 25, 'Texas');<br/>COMMIT; | INSERT INTO company (id,name,age,address) VALUES (3, 'Allen3', 25, 'Texas'); |
| ROWNUM = N                                                   | LIMIT 1 OFFSET N                                             |
| ROWNUM <= N                                                  | LIMIT N                                                      |
| ROWNUM >= N                                                  | LIMIT ALL OFFSET N                                           |
| field1 IS NULL                                               | coalesce(field1::text, '') = ''                              |
| field2 IS NOT NULL                                           | field2 IS NOT NULL AND field2::text<> ''                     |

## COMMIT for DML/DDL

**Oracle**

DML: 'COMMIT' needed.

DDL: 'COMMIT' not needed.

**PostgreSQL**

DML: 'COMMIT' needed if in transaction.

DDL: 'COMMIT' needed if in transaction.

```
BEGIN;
CREATE TABLE caudit
  ( emp_id INT NOT NULL, entry_date text NOT NULL
  );
COMMIT;
```

## TABLE

| Oracle                                                       | PostgreSQL                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ALTER TABLE company ADD gender CHAR(1);                      | ALTER TABLE company ADD gender CHAR(1);                      |
| ALTER TABLE company ADD gender CHAR(1) DEFAULT 'M';          | ALTER TABLE company ADD gender CHAR(1) DEFAULT 'M';          |
| DESC company;<br/>SELECT * FROM user_tab_columns WHERE table_name = upper('company'); | DESC company;<br/>\d company                                 |
| ALTER TABLE company MODIFY gender CHAR(2);                   | ALTER TABLE company ALTER gender TYPE CHAR(2);               |
| ALTER TABLE company MODIFY gender NOT NULL;                  | ALTER TABLE company ALTER gender SET NOT NULL;               |
| ALTER TABLE company MODIFY gender NULL;                      | ALTER TABLE company ALTER gender DROP NOT NULL;              |
| ALTER TABLE company MODIFY gender DEFAULT 'M';               | ALTER TABLE company ALTER gender SET DEFAULT 'M';            |
| ALTER TABLE company DROP PRIMARY KEY;<br />ALTER TABLE company DROP CONSTRAINT company_pkey; | ALTER TABLE company DROP CONSTRAINT company_pkey;            |
| ALTER TABLE company ADD CONSTRAINT company_pkey PRIMARY KEY (id); | ALTER TABLE company ADD CONSTRAINT company_pkey PRIMARY KEY (id); |
| ALTER TABLE company DROP COLUMN gender;                      | ALTER TABLE company DROP COLUMN gender;                      |
| TRUNCATE TABLE company;                                      | TRUNCATE TABLE company;                                      |
| DROP TABLE company;                                          | DROP TABLE company;                                          |

## SEQUENCE

After data migration, should update SEQUENCE values.

## TRIGGER

Oracle

```
CREATE OR REPLACE TRIGGER example_trigger
AFTER INSERT ON company FOR EACH ROW
BEGIN
  INSERT INTO caudit
    (emp_id, entry_date
    ) VALUES
    (:new.id, CURRENT_TIMESTAMP
    );
END;
/
DROP TRIGGER example_trigger;
```

PostgreSQL

```
CREATE OR REPLACE FUNCTION auditlogfunc() RETURNS TRIGGER LANGUAGE plpgsql AS $$
   BEGIN
      INSERT INTO caudit(emp_id, entry_date) VALUES (new.id, current_timestamp);
      RETURN new;
   END;
$$;
CREATE TRIGGER example_trigger AFTER INSERT ON company
FOR EACH ROW EXECUTE PROCEDURE auditlogfunc();
DROP FUNCTION auditlogfunc();
DROP TRIGGER example_trigger ON company;
```

## FUNCTION

Oracle

```
CREATE OR REPLACE FUNCTION cs_fmt_browser_version(v_name varchar2,
                                                  v_version varchar2)
RETURN varchar2 IS
BEGIN
    IF v_version IS NULL THEN
        RETURN v_name;
    END IF;
    RETURN v_name || '/' || v_version;
END;
/
show errors;
```

PostgreSQL

```
CREATE OR REPLACE FUNCTION cs_fmt_browser_version(v_name varchar,
                                                  v_version varchar)
RETURNS varchar AS $$
BEGIN
    IF v_version IS NULL THEN
        RETURN v_name;
    END IF;
    RETURN v_name || '/' || v_version;
END;
$$ LANGUAGE plpgsql;
```

## PROCEDURE

Oracle

```
CREATE OR REPLACE PROCEDURE cs_update_referrer_type_proc IS
    CURSOR referrer_keys IS
        SELECT * FROM cs_referrer_keys
        ORDER BY try_order;
    func_cmd VARCHAR(4000);
BEGIN
    func_cmd := 'CREATE OR REPLACE FUNCTION cs_find_referrer_type(v_host IN VARCHAR2,
                 v_domain IN VARCHAR2, v_url IN VARCHAR2) RETURN VARCHAR2 IS BEGIN';

    FOR referrer_key IN referrer_keys LOOP
        func_cmd := func_cmd ||
          ' IF v_' || referrer_key.kind
          || ' LIKE ''' || referrer_key.key_string
          || ''' THEN RETURN ''' || referrer_key.referrer_type
          || '''; END IF;';
    END LOOP;

    func_cmd := func_cmd || ' RETURN NULL; END;';

    EXECUTE IMMEDIATE func_cmd;
END;
/
show errors;
```

```
CREATE OR REPLACE PROCEDURE cs_create_job(v_job_id IN INTEGER) IS
    a_running_job_count INTEGER;
BEGIN
    LOCK TABLE cs_jobs IN EXCLUSIVE MODE;

    SELECT count(*) INTO a_running_job_count FROM cs_jobs WHERE end_stamp IS NULL;

    IF a_running_job_count > 0 THEN
        COMMIT; -- free lock
        raise_application_error(-20000,
                 'Unable to create a new job: a job is currently running.');
    END IF;

    DELETE FROM cs_active_job;
    INSERT INTO cs_active_job(job_id) VALUES (v_job_id);

    BEGIN
        INSERT INTO cs_jobs (job_id, start_stamp) VALUES (v_job_id, now());
    EXCEPTION
        WHEN dup_val_on_index THEN NULL; -- don't worry if it already exists
    END;
    COMMIT;
END;
/
show errors
```

PostgreSQL

```
CREATE OR REPLACE PROCEDURE cs_update_referrer_type_proc() AS $func$
DECLARE
    referrer_keys CURSOR IS
        SELECT * FROM cs_referrer_keys
        ORDER BY try_order;
    func_body text;
    func_cmd text;
BEGIN
    func_body := 'BEGIN';

    FOR referrer_key IN referrer_keys LOOP
        func_body := func_body ||
          ' IF v_' || referrer_key.kind
          || ' LIKE ' || quote_literal(referrer_key.key_string)
          || ' THEN RETURN ' || quote_literal(referrer_key.referrer_type)
          || '; END IF;' ;
    END LOOP;

    func_body := func_body || ' RETURN NULL; END;';

    func_cmd :=
      'CREATE OR REPLACE FUNCTION cs_find_referrer_type(v_host varchar,
                                                        v_domain varchar,
                                                        v_url varchar)
        RETURNS varchar AS '
      || quote_literal(func_body)
      || ' LANGUAGE plpgsql;' ;

    EXECUTE func_cmd;
END;
$func$ LANGUAGE plpgsql;
```

```
CREATE OR REPLACE PROCEDURE cs_create_job(v_job_id integer) AS $$
DECLARE
    a_running_job_count integer;
BEGIN
    LOCK TABLE cs_jobs IN EXCLUSIVE MODE;

    SELECT count(*) INTO a_running_job_count FROM cs_jobs WHERE end_stamp IS NULL;

    IF a_running_job_count > 0 THEN
        COMMIT; -- free lock
        RAISE EXCEPTION 'Unable to create a new job: a job is currently running';
    END IF;

    DELETE FROM cs_active_job;
    INSERT INTO cs_active_job(job_id) VALUES (v_job_id);

    BEGIN
        INSERT INTO cs_jobs (job_id, start_stamp) VALUES (v_job_id, now());
    EXCEPTION
        WHEN unique_violation THEN
            -- don't worry if it already exists
    END;
    COMMIT;
END;
$$ LANGUAGE plpgsql;
```

## PL/SQL vs PL/pgSQL

### SELECT INTO

Oracle

```
--INTO in PL/SQL
DECLARE
    myrec company%ROWTYPE;
BEGIN
    SELECT * INTO myrec from company where id=3;
END;
/
```

PostgreSQL

```
--INTO in FUNCTION(PL/pgSQL) is normal, SHOULD USE STRICT
CREATE OR REPLACE FUNCTION myid(myid INTEGER) RETURNS text LANGUAGE plpgsql AS $$
DECLARE
    myrec company%ROWTYPE;
BEGIN
    SELECT * INTO STRICT myrec from company where id=$1;
	RETURN myrec.name;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RAISE EXCEPTION 'company id % not found', myid;
        WHEN TOO_MANY_ROWS THEN
            RAISE EXCEPTION 'company id % not unique', myid;
END;
$$;
```

### PL/SQL Block

Oracle

```
BEGIN
  UPDATE company SET age=26 WHERE id=2;
  COMMIT;
END;
/
```

PostgreSQL

```
BEGIN;
UPDATE company SET age=26 WHERE id=2;
COMMIT;

BEGIN;
UPDATE company SET age=26 WHERE id=2;
END TRANSACTION;
```

### EXCEPTION

```
   ln_count := SQL%rowcount;
EXCEPTION
   WHEN OTHERS THEN
   ...dbms_utility.format_error_backtrace() || dbms_utility.format_error_stack()..
```

PostgreSQL

```
GET DIAGNOSTICS ln_count = ROW_COUNT;
EXCEPTION
	WHEN OTHERS THEN
		GET STACKED DIAGNOSTICS lv_state = returned_sqlstate,
		lv_msg = message_text,
		lv_detail = pg_exception_detail,
		lv_hint = pg_exception_hint,
		lv_context = pg_exception_context;
```

### Perform

Executing a Command with No Result

```
PERFORM query;
PERFORM create_mv('cs_session_page_requests_mv', my_query);
```

### SECURITY DEFINER

```
CREATE FUNCTION check_password(uname TEXT, pass TEXT)
RETURNS BOOLEAN AS $$
DECLARE passed BOOLEAN;
BEGIN
        SELECT  (pwd = $2) INTO passed
        FROM    pwds
        WHERE   username = $1;

        RETURN passed;
END;
$$  LANGUAGE plpgsql
    SECURITY DEFINER
    -- Set a secure search_path: trusted schema(s), then 'pg_temp'.
    SET search_path = admin, pg_temp;
```

```
BEGIN;
CREATE FUNCTION check_password(uname TEXT, pass TEXT) ... SECURITY DEFINER;
REVOKE ALL ON FUNCTION check_password(uname TEXT, pass TEXT) FROM PUBLIC;
GRANT EXECUTE ON FUNCTION check_password(uname TEXT, pass TEXT) TO admins;
COMMIT;
```

## System Functions

### TRUNC

Oracle

```
TRUNC(create_dt)
```

PostgreSQL

```
DATE_TRUNC('day', create_dt)
```

### instr

Oracle

```
instr(upper(pv_usragentstr) ,'REDHAT',1) >0
ltrim(SUBSTR(usragentstr,instr(useragentstr,'Android') +8, instr(usragentstr,';',1,3) -instr(usragentstr, 'Android') -8)) ;
```

PostgreSQL

```
POSITION('REDHAT' IN UPPER(pv_usragentstr)) > 0
TRIM(LEADING FROM REPLACE(SUBSTRING(useragentstr FROM 'Android[\s]+[\d\.]+'), 'Android', ''))
```

This section contains the code for a set of Oracle-compatible `instr` functions that you can use to simplify your porting efforts.

```
--
-- instr functions that mimic Oracle's counterpart
-- Syntax: instr(string1, string2 [, n [, m]])
-- where [] denotes optional parameters.
--
-- Search string1, beginning at the nth character, for the mth occurrence
-- of string2.  If n is negative, search backwards, starting at the abs(n)'th
-- character from the end of string1.
-- If n is not passed, assume 1 (search starts at first character).
-- If m is not passed, assume 1 (find first occurrence).
-- Returns starting index of string2 in string1, or 0 if string2 is not found.
--

CREATE FUNCTION instr(varchar, varchar) RETURNS integer AS $$
BEGIN
    RETURN instr($1, $2, 1);
END;
$$ LANGUAGE plpgsql STRICT IMMUTABLE;


CREATE FUNCTION instr(string varchar, string_to_search_for varchar,
                      beg_index integer)
RETURNS integer AS $$
DECLARE
    pos integer NOT NULL DEFAULT 0;
    temp_str varchar;
    beg integer;
    length integer;
    ss_length integer;
BEGIN
    IF beg_index > 0 THEN
        temp_str := substring(string FROM beg_index);
        pos := position(string_to_search_for IN temp_str);

        IF pos = 0 THEN
            RETURN 0;
        ELSE
            RETURN pos + beg_index - 1;
        END IF;
    ELSIF beg_index < 0 THEN
        ss_length := char_length(string_to_search_for);
        length := char_length(string);
        beg := length + 1 + beg_index;

        WHILE beg > 0 LOOP
            temp_str := substring(string FROM beg FOR ss_length);
            IF string_to_search_for = temp_str THEN
                RETURN beg;
            END IF;

            beg := beg - 1;
        END LOOP;

        RETURN 0;
    ELSE
        RETURN 0;
    END IF;
END;
$$ LANGUAGE plpgsql STRICT IMMUTABLE;


CREATE FUNCTION instr(string varchar, string_to_search_for varchar,
                      beg_index integer, occur_index integer)
RETURNS integer AS $$
DECLARE
    pos integer NOT NULL DEFAULT 0;
    occur_number integer NOT NULL DEFAULT 0;
    temp_str varchar;
    beg integer;
    i integer;
    length integer;
    ss_length integer;
BEGIN
    IF occur_index <= 0 THEN
        RAISE 'argument ''%'' is out of range', occur_index
          USING ERRCODE = '22003';
    END IF;

    IF beg_index > 0 THEN
        beg := beg_index - 1;
        FOR i IN 1..occur_index LOOP
            temp_str := substring(string FROM beg + 1);
            pos := position(string_to_search_for IN temp_str);
            IF pos = 0 THEN
                RETURN 0;
            END IF;
            beg := beg + pos;
        END LOOP;

        RETURN beg;
    ELSIF beg_index < 0 THEN
        ss_length := char_length(string_to_search_for);
        length := char_length(string);
        beg := length + 1 + beg_index;

        WHILE beg > 0 LOOP
            temp_str := substring(string FROM beg FOR ss_length);
            IF string_to_search_for = temp_str THEN
                occur_number := occur_number + 1;
                IF occur_number = occur_index THEN
                    RETURN beg;
                END IF;
            END IF;

            beg := beg - 1;
        END LOOP;

        RETURN 0;
    ELSE
        RETURN 0;
    END IF;
END;
$$ LANGUAGE plpgsql STRICT IMMUTABLE;
```

# Tool

[ora2pg](http://www.ora2pg.com) - Oracle to PostgreSQL migration tool for both data and schema. Refer to its [doc](http://www.ora2pg.com/documentation.html) for more details.

Example configuration file

```
ORACLE_HOME	/usr/lib/oracle/12.2/client64
ORACLE_DSN	dbi:Oracle:host=10.1.1.10;sid=db1;port=1521
ORACLE_USER	user1
ORACLE_PWD	user1
SCHEMA	user1
PACKAGE_AS_SCHEMA	0
EXPORT_INVALID	1
PG_NUMERIC_TYPE	0
DEFAULT_NUMERIC integer
```

Useful commands

```
ora2pg --project_base ~/alvin/app/migration/ --init_project alvin
export LD_LIBRARY_PATH="/usr/lib/oracle/12.2/client64/lib/"
export ORACLE_HOME=/usr/lib/oracle/12.2/client64
export PATH=/usr/lib/oracle/12.2/client64/bin:$PATH
./export_schema.sh | tee -a export.log
```