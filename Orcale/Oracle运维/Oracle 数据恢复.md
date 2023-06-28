
# 根据被删除日志来恢复数据

在Oracle数据库中使用Flashback Query功能可以达到这个目的：

- 首先，需要查询v$sql视图中满足条件的SQL文的SQL_ID和执行时间。可以使用以下语句：

```sql
select sql_id, first_load_time from v$sql where sql_text like 'DELETE FROM table_name%';
```

- 然后，需要根据SQL_ID和执行时间来查询被删除的数据。可以使用以下语句：

```sql
SELECT
	* 
FROM
	equip_move_dtl_inf AS OF timestamp to_timestamp( < first_load_time >, 'YYYY-MM-DD HH:MI:SS' ) 
WHERE
	ROWID IN ( SELECT ROWID FROM v$flashback_transaction_query WHERE xid = ( SELECT rawtohex( versions_xid ) FROM v$sql WHERE sql_id = < sql_id > ) );
```

- 最后，需要将被删除的数据重新插入到表中。可以使用以下语句：

```sql
INSERT INTO equip_move_dtl_inf (
	SELECT
		* 
	FROM
		equip_move_dtl_inf AS OF timestamp to_timestamp( < first_load_time >, 'YYYY-MM-DD HH:MI:SS' ) 
	WHERE
	ROWID IN ( SELECT ROWID FROM v$flashback_transaction_query WHERE xid = ( SELECT rawtohex( versions_xid ) FROM v$sql WHERE sql_id = < sql_id > ) ) 
	);
```

* 最后整合在一起使用

```sql
insert into equip_move_dtl_inf (select * from equip_move_dtl_inf as of timestamp to_timestamp ((select first_load_time from v$sql where sql_id = (select sql_id from v$sql where sql_text like 'DELETE FROM EQUIP_MOVE_DTL_INF%')), 'YYYY-MM-DD HH:MI:SS') where rowid in (select rowid from v$flashback_transaction_query where xid = (select rawtohex(versions_xid) from v$sql where sql_id = (select sql_id from v$sql where sql_text like 'DELETE FROM EQUIP_MOVE_DTL_INF%'))));
```
