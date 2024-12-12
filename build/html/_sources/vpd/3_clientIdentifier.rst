############################################
3. クライアント識別子を用いた制御
############################################


**実施内容**
+ 
+ VPD関数の作成
+ VPDのポリシーの作成
+ HRユーザーまたはSYSユーザーでEMMPLOYEES表を確認
+ SALES_APPユーザーでEMMPLOYEES表を確認

VPD
client_Identifier


****************************
APPユーザーの作成
****************************

ここでは新しくAPPユーザーを作成します。作成の流れはSALES_APPユーザーと同じです。

.. code:: sql

    CREATE USER APP IDENTIFIED BY <password> DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;


以下は実行例です。証跡としてパスワードが残らないよう、2つのコマンドに分けて設定しています。

.. code:: sql

    SQL> create user app
    2     default tablespace users
    3     temporary tablespace temp;

    -- passwordを設定
    SQL> password app
    Changing password for sales_app
    New password: <パスワードを入力>
    Retype new password: <パスワードを再入力>
    Password changed


次に、APP ユーザーにセッション作成権限を付与します。

.. code:: sql

    SQL> grant create session to app;


| さらに、HRスキーマに対して SELECT 権限を付与します。
| スキーマ単位で権限付与する方法は23aiの新機能となっており、これにより、APP ユーザーは HR スキーマのテーブルに対してデータを参照できるようになります。

.. code:: sql

    -- 23aiの新機能、スキーマ権限
    SQL> grant select any table on schema HR to app;


****************************
VPD関数の作成
****************************

jobは以下の19つあります。

SQL>  select job_id, JOB_TITLE from hr.jobs;

JOB_ID     JOB_TITLE
---------- -----------------------------------
AD_PRES    President
AD_VP      Administration Vice President
AD_ASST    Administration Assistant
FI_MGR     Finance Manager
FI_ACCOUNT Accountant
AC_MGR     Accounting Manager
AC_ACCOUNT Public Accountant
SA_MAN     Sales Manager
SA_REP     Sales Representative
PU_MAN     Purchasing Manager
PU_CLERK   Purchasing Clerk
ST_MAN     Stock Manager
ST_CLERK   Stock Clerk
SH_CLERK   Shipping Clerk
IT_PROG    Programmer
MK_MAN     Marketing Manager
MK_REP     Marketing Representative
HR_REP     Human Resources Representative
PR_REP     Public Relations Representative


ここでは、以下の3つに分けてwhere句をそれぞれつけていきたいと思います。

+ Viewer: 一般的なスタッフ職やコントリビューターを表示
+ Editor: 閲覧専用ユーザーで見られる行に加え、ミドルマネジメント職等を表示
+ Admin: 全てのJOBタイトルが閲覧可能

CREATE OR REPLACE FUNCTION hr.get_app_predicate( 
    p_schema IN VARCHAR2,
    p_table  IN VARCHAR2
    )
    RETURN VARCHAR2
    IS
        v_predicate VARCHAR2 (400);
    BEGIN
        IF SYS_CONTEXT('USERENV', 'SESSION_USER') = 'SALES_APP' THEN
            v_predicate := 'JOB_ID LIKE ''SA_%''';
        ELSIF SYS_CONTEXT('USERENV', 'SESSION_USER') = 'APP' THEN
            IF SYS_CONTEXT('USERENV', 'CLIENT_IDENTIFIER') = 'VIEWER' THEN
                v_predicate := 'REGEXP_LIKE(JOB_ID, ''_(ACCOUNT|CLERK|REP)$'')';
            ELSIF SYS_CONTEXT('USERENV', 'CLIENT_IDENTIFIER') = 'EDITOR' THEN
                v_predicate := 'NOT REGEXP_LIKE(JOB_ID, ''_(PRES|VP|ASST)$'')';
            ELSIF SYS_CONTEXT('USERENV', 'CLIENT_IDENTIFIER') = 'ADMIN' THEN
                v_predicate := '1=1';
            ELSE
                v_predicate := '1=2';
            END IF;
        ELSE
            v_predicate := '1=1';
        END IF;
    RETURN v_predicate;
END get_app_predicate;
/

閲覧専用ユーザー(Viewer): 一般的なスタッフ職やコントリビューターを表示

例:
FI_ACCOUNT (Accountant)
AC_ACCOUNT (Public Accountant)
SA_REP (Sales Representative)
PU_CLERK (Purchasing Clerk)
ST_CLERK (Stock Clerk)
SH_CLERK (Shipping Clerk)
MK_REP (Marketing Representative)
HR_REP (Human Resources Representative)
PR_REP (Public Relations Representative)
一般権限ユーザー(Editor): 閲覧専用ユーザーで見られる行に加え、ミドルマネジメント職等を表示

閲覧専用ユーザーが見られる職種＋以下職種:
FI_MGR (Finance Manager)
AC_MGR (Accounting Manager)
SA_MAN (Sales Manager)
PU_MAN (Purchasing Manager)
ST_MAN (Stock Manager)
IT_PROG (Programmer)
MK_MAN (Marketing Manager)
管理者(Admin): 全てのJOBタイトルが閲覧可能

上記全て＋
AD_PRES (President)
AD_VP (Administration Vice President)
AD_ASST (Administration Assistant)



****************************
VPDポリシーの作成
****************************

それでは先ほど作成したVPD関数を指定してVPDポリシーを作成していきます。
すでに作成したemployees_vpd_policyをDROPし、再作成する形で適用します。


.. code:: sql

ある場合はいったん削除する
BEGIN
    DBMS_RLS.DROP_POLICY (
        object_schema   => 'HR',
        object_name     => 'EMPLOYEES',
        policy_name     => 'employees_vpd_policy'
    ); 
END;
/

BEGIN
    DBMS_RLS.ADD_POLICY (
        object_schema   => 'HR',
        object_name     => 'EMPLOYEES',
        policy_name     => 'employees_vpd_policy',
        function_schema => 'HR',
        policy_function => 'get_app_predicate'
    );
END;
/


