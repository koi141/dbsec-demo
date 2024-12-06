############################################
1. SQL Firewallの準備
############################################



**実施内容**

+ SQL Firewallを有効化する
+ OCIコンソール


****************************
SQL Firewallを有効化する
****************************

ステータスを確認します。
SQL> set markup csv on
SQL> select * from DBA_SQL_FIREWALL_STATUS;
"STATUS","STATUS_UPDATED_ON","EXCLUDE_JOBS"
"DISABLED","06-DEC-24 05.50.51.263690 AM +00:00","Y"

DISABLEDなので無効化されていることが分かります。


EXEC DBMS_SQL_FIREWALL.ENABLE;

有効化されたかを確認します


SQL> select * from DBA_SQL_FIREWALL_STATUS;
"STATUS","STATUS_UPDATED_ON","EXCLUDE_JOBS"
"ENABLED","06-DEC-24 05.52.21.098865 AM +00:00","Y"

ENABLEDになったことが分かります。


********************************
SQLトラフィックをキャプチャする
********************************

SALES_APPユーザーからくるSQLのキャプチャを開始します。

BEGIN
	DBMS_SQL_FIREWALL.CREATE_CAPTURE (
		username         => 'SALES_APP',
		top_level_only   => TRUE,
		start_capture    => TRUE
	);
END;
/

PL/SQL procedure successfully completed.


SALES_APPユーザーに切り替え、適当にSQLを実行します。

$ sqlplus sales_app/Welcome1#Welcome1#@localhost:1521/freepdb1


Select * From hr.job_history;

select job_title from hr.job_history;

desc EMPLOYEEs;

Select first_name From hr.employees where employee_id IN(102,200);


select first_name, email, job_id from customers where job_id like 'SA_%';

select jfiros from aaiorwae;
