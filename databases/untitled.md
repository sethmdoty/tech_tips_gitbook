# MsSQL Kill User Session

`MSSQL kill user session`

`SELECT DB_NAME(dbid) as DBName, COUNT(dbid) as NumberOfConnections, loginame as LoginName FROM sys.sysprocesses WHERE dbid > 0 GROUP BY dbid, loginame ;`

`SELECT spid from master..sysprocesses WHERE loginame = ‘HABIB\Administrator’`

`Kill $PID`

`KILL ALL SESSIONS`

`USE master; GO ALTER DATABASE DB_NAME SET SINGLE_USER WITH ROLLBACK IMMEDIATE; ALTER DATABASE $DB_NAME SET MULTI_USER; GO`

