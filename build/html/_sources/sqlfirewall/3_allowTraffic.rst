############################################
3. SQLの許可リストを作成する
############################################

**実施内容**

+ 許可リストを作成する



********************************
許可リストを作成する
********************************

現在許可されているSQLを確認します。

    SQL> select ALLOWED_SQL_ID, SQL_TEXT from DBA_SQL_FIREWALL_ALLOWED_SQL;

    no rows selected

何も設定していないので、何も表示されないはずです、

select * from  dba_sql_firewall_allowed_ip_addr  where username = 'SALES_APP';

キャプチャしたSQLログを使用して、SQL許可リストを作成します。

EXEC DBMS_SQL_FIREWALL.GENERATE_ALLOW_LIST ('SALES_APP');

もう一度、許可リストを確認すると許可されたSQLが確認できるはずです。

SQL> select ALLOWED_SQL_ID, SQL_TEXT from DBA_SQL_FIREWALL_ALLOWED_SQL;
"ALLOWED_SQL_ID","SQL_TEXT"
1,"SELECT DECODE (USER,:""SYS_B_0"",XS_SYS_CONTEXT (:""SYS_B_1"",:""SYS_B_2""),USER) FROM SYS.DUAL"
5,"SELECT FIRST_NAME FROM HR.EMPLOYEES WHERE EMPLOYEE_ID IN (:""SYS_B_0"",:""SYS_B_1"")"
2,"SELECT * FROM HR.JOB_HISTORY"
6,"SELECT FIRST_NAME,EMAIL,JOB_ID FROM HR.EMPLOYEES WHERE JOB_ID LIKE :""SYS_B_0"""
3,"DESCRIBE HR.JOB_HISTORY"
4,"SELECT JOB_ID FROM HR.JOB_HISTORY"

6 rows selected.


********************************
許可リストを有効化する
********************************
許可リストを有効化します。

BEGIN
	DBMS_SQL_FIREWALL.ENABLE_ALLOW_LIST(
		username => 'SALES_APP',
		enforce  => DBMS_SQL_FIREWALL.ENFORCE_ALL,
		block    => TRUE
		);
END;
/

SQL Firewallでは、blockオプションの設定に関係なく、一致しないデータベース接続またはSQL文の違反ログが常に生成されます。

