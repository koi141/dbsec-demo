############################################
2. OLSポリシーの準備
############################################


**実施内容**

+ レベル作成
+ ラベル作成
+ 表にOLSを適用
+ データに対してラベルを適用


****************************
レベルを作成
****************************
レベルを作成します

https://docs.oracle.com/cd/G11854_01/olsag/oracle-label-security-pl-sql-packages.html#GUID-C266B7C5-DAF4-4A97-B17F-AF39D286F17D

BEGIN
   SA_COMPONENTS.CREATE_LEVEL (
      policy_name => 'OLS_POL_DEMO',
      level_num   => 4000,
      short_name  => 'SENS',
      long_name   => 'SENSITIVE');

   SA_COMPONENTS.CREATE_LEVEL (
      policy_name => 'OLS_POL_DEMO',
      level_num   => 3000,
      short_name  => 'CONF',
      long_name   => 'CONFIDENTIAL');
      
   SA_COMPONENTS.CREATE_LEVEL (
      policy_name => 'OLS_POL_DEMO',
      level_num   => 2000,
      short_name  => 'INTL',
      long_name   => 'INTERNAL');
END;
/



作成したレベルを確認します

SQL> select * from ALL_SA_LEVELS;
"POLICY_NAME","LEVEL_NUM","SHORT_NAME","LONG_NAME"
"OLS_POL_DEMO",3000,"CONF","CONFIDENTIAL"
"OLS_POL_DEMO",2000,"INTL","INTERNAL"
"OLS_POL_DEMO",4000,"SENS","SENSITIVE"



****************************
データ・ラベルを作成
****************************
ラベルを作成します

どのようなラベルを設定するかを作成します。
今回作成するラベルは先ほど作成したレベルしか用いませんので、先ほどのレベルと同じ内容です

https://docs.oracle.com/cd/G11854_01/olsag/oracle-label-security-pl-sql-packages.html#GUID-6EE1DD6A-C893-4C01-88A9-C1AE36F224E3

BEGIN
	SA_LABEL_ADMIN.CREATE_LABEL (
		policy_name  => 'OLS_POL_DEMO',
		label_tag    => 400,
		label_value  => 'SENS',
		data_label   => TRUE);
			
	SA_LABEL_ADMIN.CREATE_LABEL (
		policy_name  => 'OLS_POL_DEMO',
		label_tag    => 300,
		label_value  => 'CONF',
		data_label   => TRUE);
	
	SA_LABEL_ADMIN.CREATE_LABEL (
		policy_name  => 'OLS_POL_DEMO',
		label_tag    => 200,
		label_value  => 'INTL',
		data_label   => TRUE);
END;
/

確認します

SQL> select * from ALL_SA_LABELS;
"POLICY_NAME","LABEL","LABEL_TAG","LABEL_TYPE"
"OLS_POL_DEMO","SENS",400,"USER/DATA LABEL"
"OLS_POL_DEMO","CONF",300,"USER/DATA LABEL"
"OLS_POL_DEMO","INTL",200,"USER/DATA LABEL"

****************************
ユーザー・ラベルを作成
****************************

ユーザーに閲覧できるレベルの幅を割り当てます。

BEGIN
    SA_USER_ADMIN.SET_LEVELS (
        policy_name  => 'OLS_POL_DEMO',
        user_name    => 'HR', 
        max_level    => 'SENS',
        min_level    => 'INTL');

    SA_USER_ADMIN.SET_LEVELS (
        policy_name  => 'OLS_POL_DEMO',
        user_name    => 'SALES_APP', 
        max_level    => 'CONF',
        min_level    => 'INTL');
END;
/

****************************
表にポリシーを適用する
****************************

作成したOLSポリシーをhr.job_history_4ols表に適用します。

BEGIN
    SA_POLICY_ADMIN.APPLY_TABLE_POLICY (
        policy_name    => 'OLS_POL_DEMO',
        schema_name    => 'HR', 
        table_name     => 'JOB_HISTORY_4OLS',
        table_options  => 'READ_CONTROL');
END;
/

READ_CONTROLは、ユーザーが後で実行するすべての問合せにポリシーの施行を適用します。
SELECT、UPDATE文とDELETE文の処理にはかならず適用されます。

この時点でJOB_HISTORY_4OLS表を見てみましょう。

    SQL> desc hr.JOB_HISTORY_4OLS;
    Name                                      Null?    Type
    ----------------------------------------- -------- ----------------------------
    EMPLOYEE_ID                               NOT NULL NUMBER(6)
    START_DATE                                NOT NULL DATE
    END_DATE                                  NOT NULL DATE
    JOB_ID                                    NOT NULL VARCHAR2(10)
    DEPARTMENT_ID                                      NUMBER(4)
    OLS_LABEL_DEMO                                     NUMBER(10)

OLS_LABEL_DEMO列が追加され、NUMBER型であることが確認できます。

OLSポリシーでの制御を有効化します。

BEGIN
    SA_POLICY_ADMIN.ENABLE_TABLE_POLICY (
        policy_name => 'OLS_POL_DEMO',
        schema_name => 'HR',
        table_name  => 'JOB_HISTORY_4OLS');
END;
/

.. SQL> select * from DBA_SA_POLICIES;
.. "POLICY_NAME","COLUMN_NAME","STATUS","POLICY_OPTIONS","POLICY_SUBSCRIBED"
.. "OLS_POL_DEMO","OLS_LABEL_DEMO","ENABLED",,"FALSE"
.. どこかに確認を入れたい


CHAR_TO_LABELファンクションはラベル文字列をラベル（数字）に変換します。
対して、LABEL_TO_CHARファンクションはラベル（数字）を文字列に戻します。



****************************
データにラベルを付けていく
****************************

今回はJOB_ID が
AC_MGR や SA_MAN のレコードは SENSITIVE ラベルを設定します。


UPDATE HR.JOB_HISTORY_4OLS
SET    OLS_LABEL_DEMO = CHAR_TO_LABEL('OLS_POL_DEMO','SENS')
WHERE  JOB_ID LIKE '%_MGR' 
OR     JOB_ID LIKE '%_MAN';


2 rows updated. されます。



JOB_ID が AC_ACCOUNT, MK_REP のレコードに対しては CONFIDENTIAL ラベルを設定します。

UPDATE HR.JOB_HISTORY_4OLS
SET    OLS_LABEL_DEMO = CHAR_TO_LABEL('OLS_POL_DEMO','CONF')
WHERE  JOB_ID LIKE '%_ACCOUNT' 
OR     JOB_ID LIKE '%_REP';

4 rows updated. されます。

IT_PROG または ST_CLERK のレコードは INTERNAL に分類。
UPDATE HR.JOB_HISTORY_4OLS
SET    OLS_LABEL_DEMO = CHAR_TO_LABEL('OLS_POL_DEMO','INTL')
WHERE  JOB_ID LIKE '%_PROG' 
OR     JOB_ID LIKE '%_CLERK'
OR     JOB_ID LIKE '%_ASST';

4 rows updated. されます。

最後にcommitをします。

SQL> commit;

Commit complete.


正しく結果が反映されたことを確認します。

SQL> select JOB_ID, OLS_LABEL_DEMO from HR.JOB_HISTORY_4OLS;
"JOB_ID","OLS_LABEL_DEMO"
"IT_PROG",200
"AC_ACCOUNT",300
"AC_MGR",400
"MK_REP",300
"ST_CLERK",200
"ST_CLERK",200
"AD_ASST",200
"SA_REP",300
"SA_MAN",400
"AC_ACCOUNT",300

10 rows selected.



