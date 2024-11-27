##########################################
Data Redactionを試す
##########################################


*****************************************
SALES_APPユーザーを作成
*****************************************

HRユーザーと結果を比較するため、マスキングされたデータを返すユーザーとしてSALES_APPユーザーを作成します。


.. code:: sql

    create user SALES_APP
        DEFAULT TABLESPACE users
        TEMPORARY TABLESPACE temp

    SQL> create user sales_app
    2     default tablespace users
    3     temporary tablespace temp;

    User created.

    -- passwordを設定
    SQL> password sales_app
    Changing password for sales_app
    New password:
    Retype new password:
    Password changed

    SQL> grant create session to sales_app;

    Grant succeeded.

    -- 23aiの新機能、スキーマ権限
    SQL> grant select any table on schema HR to sales_app;

    Grant succeeded.



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




.. code-block:: sql
    :caption: HRユーザーで接続

    $ sqlplus hr@localhost:1521/freepdb1

    SQL> select first_name, salary, commission_pct from employees;
    ...
    FIRST_NAME               SALARY COMMISSION_PCT
    -------------------- ---------- --------------
    Peter                      2500
    John                      14000             .4
    Karen                     13500             .3
    Alberto                   12000             .3
    Gerald                    11000             .3
    Eleni                     10500             .2
    ...
    107 rows selected.



.. code-block:: sql
    :caption: SALES_APPユーザーで接続

    $ sqlplus sales_app@localhost:1521/freepdb1

    select first_name, salary, commission_pct from hr.employees;
    ...
    FIRST_NAME               SALARY COMMISSION_PCT
    -------------------- ---------- --------------
    Peter                         0
    John                          0              0
    Karen                         0              0
    Alberto                       0              0
    Gerald                        0              0
    Eleni                         0              0
    ...
    107 rows selected.



    select first_name, salary, commission_pct from hr.employees where salary > 10000;

    SELECT employee_id, first_name, last_name, salary FROM hr.employees 
        WHERE salary > (SELECT AVG(salary) FROM hr.employees);
