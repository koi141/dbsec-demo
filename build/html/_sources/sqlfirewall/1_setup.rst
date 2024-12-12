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


