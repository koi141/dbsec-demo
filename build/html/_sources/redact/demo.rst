##########################################
Data Redactionを試す
##########################################


*****************************************
SALES_APPユーザーを作成
*****************************************

HRユーザーと結果を比較するため、マスキングされたデータを返すユーザーとしてSALES_APPユーザーを作成します。


.. code:: sql

    create user SALES_APP identified by


*****************************************
リダクションポリシーを作成する
*****************************************




SALES_APPユーザーには salary列とcommission_pct列をマスキングするポリシーを作成します。


一度の関数の実行で、一つの列にしか適用できないため、salary列とcommission_pct列に適用するポリシーは別途実行していきます。


.. code:: sql

BEGIN
    DBMS_REDACT.ADD_POLICY(
        object_schema => 'HR',
        object_name => 'EMPLOYEES',
        column_name => 'SALARY',
        policy_name => 'POL_REDCT_EMPLOYEES_SALARY',
        function_type => DBMS_REDACTION.CONSTANT,
        function_parameters => 0,
        expression => 'SYS_CONTEXT(''USERENV'', ''SESSION_USER'') = ''SALES_APP'''
    );
END;
/


.. code:: sql

    SQL> BEGIN
    2     DBMS_REDACT.ADD_POLICY(
    3             object_schema => 'HR',
    4             object_name => 'EMPLOYEES',
    5             column_name => 'SALARY',
    6             policy_name => 'POL_REDCT_EMPLOYEES_SALARY',
    7             function_type => DBMS_REDACT.FULL,
    8             expression => 'SYS_CONTEXT(''USERENV'', ''SESSION_USER'') = ''SALES_APP'''
    9     );
    10  END;
    11  /

    PL/SQL procedure successfully completed.

    -- ポリシーの作成を確認
    SQL> select * from redaction_policies;
    "OBJECT_OWNER","OBJECT_NAME","POLICY_NAME","EXPRESSION","ENABLE","POLICY_DESCRIPTION"
    "HR","EMPLOYEES","POL_REDCT_EMPLOYEES_SALARY","SYS_CONTEXT('USERENV', 'SESSION_USER') = 'SALES_APP'","YES",


BEGIN
    DBMS_REDACT.ALTER_POLICY(
        object_schema => 'HR',
        object_name => 'EMPLOYEES',
        policy_name => 'POL_REDCT_EMPLOYEES_SALARY',
        action => DBMS_REDACT.ADD_COLUMN,
        column_name => 'COMMISSION_PCT',
        function_type => DBMS_REDACT.FULL
    );
END;
/

SQL> BEGIN
  2     DBMS_REDACT.ALTER_POLICY(
  3             object_schema => 'HR',
  4             object_name => 'EMPLOYEES',
  5             policy_name => 'POL_REDCT_EMPLOYEES_SALARY',
  6             action => DBMS_REDACT.ADD_COLUMN,
  7             column_name => 'COMMISSION_PCT',
  8             function_type => DBMS_REDACT.FULL
  9     );
 10  END;
 11  /

PL/SQL procedure successfully completed.

-- ポリシーの作成を確認


-- リダクション対象の列を確認
SQL> select OBJECT_OWNER, OBJECT_NAME, COLUMN_NAME, FUNCTION_TYPE from redaction_columns;
"OBJECT_OWNER","OBJECT_NAME","COLUMN_NAME","FUNCTION_TYPE"
"HR","EMPLOYEES","SALARY","FULL REDACTION"
"HR","EMPLOYEES","COMMISSION_PCT","FULL REDACTION"