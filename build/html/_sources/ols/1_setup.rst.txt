############################################
1. Oracle Label Securityの準備
############################################


**実施内容**

+ 表の準備
+ OLSの設定の確認



****************************
表の準備
****************************
ラベルを設定する表を別で作成してみます。

今回はHRスキーマのJOB_HISTORY_LABELを使用することにします。

SQL> CREATE TABLE hr.job_history_4ols AS SELECT * FROM hr.job_history WHERE 1=0;  2

Table created.


SQL> INSERT INTO hr.job_history_4ols SELECT * FROM hr.job_history;

10 rows created.

表ができたことを確認します

SQL> select * from hr.job_history_4ols;

EMPLOYEE_ID START_DAT END_DATE  JOB_ID     DEPARTMENT_ID
----------- --------- --------- ---------- -------------
        102 13-JAN-11 24-JUL-16 IT_PROG               60
        101 21-SEP-07 27-OCT-11 AC_ACCOUNT           110
        101 28-OCT-11 15-MAR-15 AC_MGR               110
        201 17-FEB-14 19-DEC-17 MK_REP                20
        114 24-MAR-16 31-DEC-17 ST_CLERK              50
        122 01-JAN-17 31-DEC-17 ST_CLERK              50
        200 17-SEP-05 17-JUN-11 AD_ASST               90
        176 24-MAR-16 31-DEC-16 SA_REP                80
        176 01-JAN-17 31-DEC-17 SA_MAN                80
        200 01-JUL-12 31-DEC-16 AC_ACCOUNT            90

10 rows selected.


insert into HR.JOB_HISTORY_LABEL
  nologging
    select PROD_ID,
           CUST_ID,
           TIME_ID+15*365+3, -- 年月日のデータを15年間進める
           CHANNEL_ID,
           PROMO_ID,
           QUANTITY_SOLD,
           AMOUNT_SOLD,
           rpad(to_char(mod(CUST_ID,30)),100,'dummy1'),
           rpad(to_char(mod(CUST_ID,30)),110,'dummy2')
      from SH.JOB_HISTORY;
commit;


****************************
設定の確認
****************************

OLSが構成されているかを確認します。DBA_SA_USER_PRIVSデータ・ディクショナリ・ビューを使用

SQL> col status for a20
SQL> col DESCRIPTION for a50
SQL> set lines 100
SQL> SELECT * FROM DBA_OLS_STATUS;

NAME                 STATUS               DESCRIPTION
-------------------- -------------------- --------------------------------------------------
OLS_CONFIGURE_STATUS FALSE                Determines if OLS is configured
OLS_ENABLE_STATUS    FALSE                Determines if OLS is enabled

ステータスの意味はそれぞれ以下の通りです。

OLS_CONFIGURE_STATUS: Oracle Label Securityが構成されているかどうかを判断します。
OLS_ENABLE_STATUS: Oracle Label Securityが有効化されているかどうかを判断します。


CDBから各PDBの設定を見るには以下のようにします

SQL> set lines 200
SQL> SELECT * FROM CDB_OLS_STATUS;

NAME                 STATUS               DESCRIPTION                                            CON_ID
-------------------- -------------------- -------------------------------------------------- ----------
OLS_CONFIGURE_STATUS FALSE                Determines if OLS is configured                             1
OLS_ENABLE_STATUS    FALSE                Determines if OLS is enabled                                1
OLS_CONFIGURE_STATUS FALSE                Determines if OLS is configured                             3
OLS_ENABLE_STATUS    FALSE                Determines if OLS is enabled                                3

****************************
OLSを有効化する
****************************

有効化していきます

SQL> show user con_name
USER is "SYS"

CON_NAME
------------------------------
FREEPDB1

SQL> EXEC LBACSYS.CONFIGURE_OLS;

PL/SQL procedure successfully completed.

Oracle Label Securityをデータベースに登録する

OLS機能のインストール、初期設定、および必要なDBオブジェクトの作成が含まれる。

SQL>  SELECT * FROM DBA_OLS_STATUS;

NAME                 STATUS               DESCRIPTION
-------------------- -------------------- --------------------------------------------------
OLS_CONFIGURE_STATUS TRUE                 Determines if OLS is configured
OLS_ENABLE_STATUS    FALSE                Determines if OLS is enabled


SQL> EXEC LBACSYS.OLS_ENFORCEMENT.ENABLE_OLS;

PL/SQL procedure successfully completed.

ENABLE_OLSプロシージャは、OLSポリシーの施行を有効にします。これにより、設定されたラベルセキュリティポリシーがデータベース内で実際に適用され、ユーザーアクセスの制御が開始されます。


SQL> SELECT * FROM DBA_OLS_STATUS;

NAME                 STATUS               DESCRIPTION
-------------------- -------------------- --------------------------------------------------
OLS_CONFIGURE_STATUS TRUE                 Determines if OLS is configured
OLS_ENABLE_STATUS    TRUE                 Determines if OLS is enabled


権限を付与されたユーザーの確認
LBAC_DBAはSA_SYSDBA PL/SQLパッケージの使用権限を提供する、事前定義ロールです

SQL> set markup csv on
SQL>  select * from dba_role_privs where granted_role = 'LBAC_DBA';
"GRANTEE","GRANTED_ROLE","ADMIN_OPTION","DELEGATE_OPTION","DEFAULT_ROLE","COMMON","INHERITED"
"SYS","LBAC_DBA","YES","NO","YES","YES","YES"
"LBACSYS","LBAC_DBA","YES","NO","YES","YES","YES"

設定を完全に反映させるためにPDBの再起動を行います。

SQL> alter pluggable database freepdb1 close immediate;

Pluggable database altered.

SQL> alter pluggable database freepdb1 open;

Pluggable database altered.


****************************
ポリシーを有効化する
****************************



新規のOracle Label Securityポリシーを作成し、
ポリシー固有の列名を定義して、デフォルト・ポリシー・オプションを指定します。

ポリシーを作成すると、そのロールが作成され、ユーザーに付与されます。
ラベルやセキュリティポリシーを管理するための論理的なグループである「ポリシーコンテナ」を作成します。
ポリシーコンテナは、これらのポリシーとラベル、そしてその他の関連する設定を論理的にまとめたものです。つまり、ポリシーコンテナは、データに付加されるラベルや、これらラベルに基づくアクセス制御ルールの適用方法を一元管理する「枠組み」を提供します。

主にラベルとポリシーから構成されます。


BEGIN
    SA_SYSDBA.CREATE_POLICY (
        policy_name      => 'OLS_POL_DEMO',
        column_name      => 'OLS_LABEL_DEMO');
END;
/

sysユーザーで構成しようとすると以下のエラーがでます

ORA-06598: insufficient INHERIT PRIVILEGES privilege
ORA-06512: at "LBACSYS.SA_SYSDBA", line 1
ORA-06512: at line 2
Help: https://docs.oracle.com/error-help/db/ora-06598/

LABACSYSユーザーでPL/SQLの権限

「実行者権限プログラム」とは、プログラムを実行したユーザーの権限で実行される PL/SQL プログラムのこと

PL/SQLは所有者の権限で実行されるため、強い権限を持つユーザーAのプログラムの実行権限を他ユーザーBに与えてしまった際、BはAの持っている強力な権限を使って処理を実行することができてしまいます。これは非常に危険で、誤った設定により予期しない権限を第三者に与える可能性が出てきます。
Oracle のデフォルトの仕組みでは、プログラムの所有者が「実行してもいいよ」と一方的に許可するだけで実行が可能になります。

そこで12cより、INHERIT PRIVILEGES権限が登場しました。この権限は、どのスキーマの「実行者権限」プログラムを実行してもよいかを、実行ユーザー側で制御するためのものです。

SYS ユーザーが LBACSYS の権限を継承してそのパッケージを利用することを許可する設定（INHERIT PRIVILEGES）が明示的に付与されていないため、エラーが発生しているのです。

INHERIT PRIVILEGES はユーザーが他のユーザーから権限を「継承」して実行するかどうかを制御するためのもので、特に高権限のユーザーに対するセキュリティ強化のために導入されました。
デフォルトでは、INHERIT PRIVILEGES 権限は PUBLIC に付与されているため、すべてのユーザーが他のユーザーのプログラムを継承して実行できる設定になっていますが、高権限ユーザーに対してはこの権限が制限されています。

SYSユーザーにはPUBLICからSYSにINHERIT PRIVILEGES 権限が付与されていないことが分かります。

    SQL> SELECT grantee, grantor, privilege FROM   DBA_tab_privs WHERE  privilege = 'INHERIT PRIVILEGES';
    "GRANTEE","GRANTOR","PRIVILEGE"
    "GSMADMIN_INTERNAL","SYS","INHERIT PRIVILEGES"
    "CTXSYS","SYS","INHERIT PRIVILEGES"
    "XDB","SYS","INHERIT PRIVILEGES"
    "PUBLIC","PUBLIC","INHERIT PRIVILEGES"
    "CTXSYS","GSMADMIN_INTERNAL","INHERIT PRIVILEGES"
    "XDB","GSMADMIN_INTERNAL","INHERIT PRIVILEGES"
    "GSMADMIN_INTERNAL","GSMCATUSER","INHERIT PRIVILEGES"
    "CTXSYS","XDB","INHERIT PRIVILEGES"
    "PUBLIC","HR","INHERIT PRIVILEGES"
    "PUBLIC","SALES_APP","INHERIT PRIVILEGES"
    "PUBLIC","LBAC_TRIGGER","INHERIT PRIVILEGES"
    "PUBLIC","XS$NULL","INHERIT PRIVILEGES"


そのため、SYS ユーザーはデフォルトでは他のスキーマが所有する「実行者権限」のプログラムを実行するための継承権限を持っていないことがわかります。

以下が確認できます:

PUBLIC に対しては複数のユーザー（例: HR, SALES_APP, LBAC_TRIGGER など）から INHERIT PRIVILEGES が付与されていますが、SYS に対してはそのような付与が行われていません。
これは、SYS ユーザーのような非常に強力な権限を持つユーザーが、他のユーザーの持つプログラムを無制限に実行してしまうことを防ぐためです。
このため、SYS ユーザーで LBACSYS の CREATE_POLICY を実行しようとした場合、INHERIT PRIVILEGES が付与されていないためにエラーが発生します。
このセキュリティ設定により、権限の強いユーザーであっても、明示的な許可なしには他のスキーマの権限を使用できないようにすることで、データベースの安全性が向上しています。

SYS ユーザーに対して LBACSYS からの INHERIT PRIVILEGES 権限を付与します。以下のSQLを使って付与できます。

    GRANT INHERIT PRIVILEGES ON USER SYS TO LBACSYS


結果の ``Grant succeeded.`` を確認し、再度実行します。

    BEGIN
        SA_SYSDBA.CREATE_POLICY (
            policy_name      => 'OLS_POL_DEMO',
            column_name      => 'OLS_LABEL_DEMO');
    END;
    /

``PL/SQL procedure successfully completed.`` が表示され、無事実行できるはずです。

https://connor-mcdonald.com/2018/10/31/the-strange-place-for-inherit-privileges/


取り消す場合は以下です
    REVOKE INHERIT PRIVILEGES ON USER SYS FROM LBACSYS;


ではポリシーを有効にします。

    EXEC SA_SYSDBA.ENABLE_POLICY ('OLS_POL_DEMO');



