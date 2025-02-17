#Logging

## Effects of logging 
Logging impacts on performance and stability

- general (text, table)
- slow-query (text, table)
- error (text)


## Binary Logging
Binary logs trace every change requested to the database.

[Binary Logging][./files/mysql-binary-logging.png]

  - archived on filesystem

        ls /var/lib/mysql/*-bin.*


  - managed  via mysql and mysqladmin
  - accessed with mysqlbinlog


## Managing Binary Logs
Binary logs are written in the following files:

        SHOW BINARY LOGS; 
        | Log_name          | File_size |
        | fabric-bin.000001 |     69414 |
        | fabric-bin.000002 |   1268759 |
  
You can show their content with 

        SHOW BINLOG EVENTS;


## Managing Binary Logs
Use a new log file

        FLUSH BINARY LOGS;
        SHOW BINARY LOGS;

or purge them 
        
        -- NOW() won't remove the last one, FLUSH it before!
        PURGE BINARY LOGS BEFORE NOW(); 
        
        -- You can use SUBDATE to simplify expiration
        PURGE BINARY LOGS BEFORE SUBDATE(CURRENT_DATE, 1); -- yesterday

        -- full clean
        RESET MASTER;s
    

## mysqlbinlog
Inspect binary logs with mysqlbinlog

        mysqlbinlog /var/lib/mysql/hostname-bin.000001
        ...
        SET @@SESSION.GTID_NEXT= '4dc24b1f-ed7d-11e4-94d2-0242ac110035:500'/*!*/;
        create database test
        ...
        SET @@SESSION.GTID_NEXT= '4dc24b1f-ed7d-11e4-94d2-0242ac110035:501'/*!*/;
        create table test.t(i int)

And replay them on another server for:
 
  - PITR 
  - replay load
  - backup.
  
## mysqlbinlog
mysqlbinlog output generates SQL files that can be replayed on other servers.

        mysqlbinlog hostname-bin.* | mysql
        
You can specify an interval on mysqlbinlog.

        mysqlbinlog hostname-bin.* \
            --start-date="2015-05-25 15:00:00" \
            --end-date="2015-05-25 15:01:00"  | wc -l
            
Exercise: modify ``--[start|end]``` -date and check that the output lines vary.
Exercise: drop a table and use mysqlbinlog data to restore it.
