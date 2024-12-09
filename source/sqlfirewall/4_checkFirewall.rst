############################################
4. SQL Firewallを体験する
############################################

**実施内容**

+ 許可リストを作成する


********************************
SALES_APPユーザーから接続し、ブロックされていることを確認する
********************************

許可リストを表示しておきます

    SQL> select ALLOWED_SQL_ID, SQL_TEXT from DBA_SQL_FIREWALL_ALLOWED_SQL;
    "ALLOWED_SQL_ID","SQL_TEXT"
    1,"SELECT DECODE (USER,:""SYS_B_0"",XS_SYS_CONTEXT (:""SYS_B_1"",:""SYS_B_2""),USER) FROM SYS.DUAL"
    5,"SELECT FIRST_NAME FROM HR.EMPLOYEES WHERE EMPLOYEE_ID IN (:""SYS_B_0"",:""SYS_B_1"")"
    2,"SELECT * FROM HR.JOB_HISTORY"
    6,"SELECT FIRST_NAME,EMAIL,JOB_ID FROM HR.EMPLOYEES WHERE JOB_ID LIKE :""SYS_B_0"""
    3,"DESCRIBE HR.JOB_HISTORY"
    4,"SELECT JOB_ID FROM HR.JOB_HISTORY"


SALES_APPユーザーで接続し、許可リストの中にあるSQLで数個問合せを行い、無事通ることを確認してみてください。

    SQL> SELECT JOB_ID FROM HR.JOB_HISTORY;

    JOB_ID
    ----------
    AC_ACCOUNT
    AC_ACCOUNT
    ...

次に許可リストにない、SQLを通してみます


SQL> SELECT JOB_ID, EMPLOYEE_ID FROM HR.JOB_HISTORY;
SELECT JOB_ID, EMPLOYEE_ID FROM HR.JOB_HISTORY
                                   *
ERROR at line 1:
ORA-47605: SQL Firewall violation
Help: https://docs.oracle.com/error-help/db/ora-47605/

このようにブロックされることがわかります。


********************************
違反ログを確認
********************************

違反ログを確認します。

SQL> set markup csv on
SQL> select USERNAME, SQL_TEXT, IP_ADDRESS, CLIENT_PROGRAM, OS_USER from DBA_SQL_FIREWALL_VIOLATIONS;
"USERNAME","SQL_TEXT","IP_ADDRESS","CLIENT_PROGRAM","OS_USER"
"SALES_APP","SELECT JOB_ID,EMPLOYEE_ID FROM HR.JOB_HISTORY","127.0.0.1","sqlplus@db-vm23ai (TNS V1-V3)","oracle"


********************************
許可リストから許可SQLを削除する
********************************

SQL許可リストを確認。IDを取得する。

SQL> select ALLOWED_SQL_ID,SQL_TEXT from DBA_SQL_FIREWALL_ALLOWED_SQL;
"ALLOWED_SQL_ID","SQL_TEXT"
1,"SELECT DECODE (USER,:""SYS_B_0"",XS_SYS_CONTEXT (:""SYS_B_1"",:""SYS_B_2""),USER) FROM SYS.DUAL"
5,"SELECT FIRST_NAME FROM HR.EMPLOYEES WHERE EMPLOYEE_ID IN (:""SYS_B_0"",:""SYS_B_1"")"
2,"SELECT * FROM HR.JOB_HISTORY"
6,"SELECT FIRST_NAME,EMAIL,JOB_ID FROM HR.EMPLOYEES WHERE JOB_ID LIKE :""SYS_B_0"""
3,"DESCRIBE HR.JOB_HISTORY"
4,"SELECT JOB_ID FROM HR.JOB_HISTORY"

6 rows selected.

ここでは3を指定して、JOB_HISTORY表のメタデータを見れないようにします。

指定したエントリを削除します。
BEGIN
	DBMS_SQL_FIREWALL.DELETE_ALLOWED_SQL (
		username       => 'SALES_APP',
		allowed_sql_id => 3
		);
END;
/

許可リストから削除されていることが分かります。

SQL> select ALLOWED_SQL_ID,SQL_TEXT from DBA_SQL_FIREWALL_ALLOWED_SQL;
"ALLOWED_SQL_ID","SQL_TEXT"
1,"SELECT DECODE (USER,:""SYS_B_0"",XS_SYS_CONTEXT (:""SYS_B_1"",:""SYS_B_2""),USER) FROM SYS.DUAL"
5,"SELECT FIRST_NAME FROM HR.EMPLOYEES WHERE EMPLOYEE_ID IN (:""SYS_B_0"",:""SYS_B_1"")"
2,"SELECT * FROM HR.JOB_HISTORY"
6,"SELECT FIRST_NAME,EMAIL,JOB_ID FROM HR.EMPLOYEES WHERE JOB_ID LIKE :""SYS_B_0"""
4,"SELECT JOB_ID FROM HR.JOB_HISTORY"



ここまでの一連の流れはすべてOCIサービスであるData Safeから行うことができます。手順5からはdata safeを用いてSQL Firewallを設定していきます。

