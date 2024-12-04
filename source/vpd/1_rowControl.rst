###########################
1. VPDで行制御を行う
###########################

**実施内容**

+ VPD関数の作成
+ VPDのポリシーの作成
+ HRユーザーまたはSYSユーザーでEMMPLOYEES表を確認
+ SALES_APPユーザーでEMMPLOYEES表を確認


****************************
VPD関数の作成
****************************

HRスキーマに、SALES_APPユーザーであればSALESの従業員のみを表示させるようにします。

VPDポリシーの対象列としては、外部キーであるJOB_IDがありますのでこれを使用することにします。

.. code:: sql

    SQL> desc hr.employees;
    Name                                      Null?    Type
    ----------------------------------------- -------- -----------------------
    EMPLOYEE_ID                               NOT NULL NUMBER(6)
    FIRST_NAME                                         VARCHAR2(20)
    LAST_NAME                                 NOT NULL VARCHAR2(25)
    EMAIL                                     NOT NULL VARCHAR2(25)
    PHONE_NUMBER                                       VARCHAR2(20)
    HIRE_DATE                                 NOT NULL DATE
    JOB_ID                                    NOT NULL VARCHAR2(10)
    ...


VPD関数を作成します。
SALES_APPユーザーであればJOB_IDがSA_から始まる行のみを取り出すようにwhere句の述語を設定します。

.. code:: sql

CREATE OR REPLACE FUNCTION hr.get_sales_predicate( 
    p_schema IN VARCHAR2,
    p_table  IN VARCHAR2
    )
    RETURN VARCHAR2
    IS
        v_predicate VARCHAR2 (400);
    BEGIN
        IF SYS_CONTEXT('USERENV', 'SESSION_USER') = 'SALES_APP' THEN
            v_predicate := 'JOB_ID LIKE ''SA_%''';
        ELSE
            v_predicate := '1=1';
        END IF;
        RETURN v_predicate;
    END get_sales_predicate;
/

実行し、 ``Function created.`` が表示されることを確認します。

****************************
VPDポリシーの作成
****************************

それでは先ほど作成したVPD関数を指定してVPDポリシーを作成していきます。
実行しているADD_POLICYプロシージャのそのほかのパラメータや詳細は `こちらから <https://docs.oracle.com/cd/F19136_01/arpls/DBMS_RLS.html#GUID-1E528A51-DE53-4961-8770-C53924E427CC>`__ ご確認ください。

.. code:: sql

    BEGIN
    DBMS_RLS.ADD_POLICY (
        object_schema   => 'HR',
        object_name     => 'EMPLOYEES',
        policy_name     => 'employees_vpd_policy',
        function_schema => 'HR',
        policy_function => 'get_sales_predicate'
    );
    END;
    /

実行し、 ``PL/SQL procedure successfully completed.`` が表示されることを確認します。

最後に作成したVPDポリシーを確認します。
VPDポリシーは ``ALL_POLICIES`` ディクショナリビューから確認することができます。

.. code:: sql

    SQL> select object_owner, object_name, policy_name, function, sel, ins, upd, del, idx, policy_type, common from all_policies where object_owner  = 'HR';
    "OBJECT_OWNER","OBJECT_NAME","POLICY_NAME"         ,"FUNCTION"           ,"SEL","INS","UPD","DEL","IDX","POLICY_TYPE","COMMON"
    "HR"          ,"EMPLOYEES"  ,"EMPLOYEES_VPD_POLICY","GET_SALES_PREDICATE","YES","NO" ,"YES","YES","NO" ,"DYNAMIC"    ,"NO"


各列の説明は以下のとおりです。

===============  ============================================================
列名              説明 
===============  ============================================================
OBJECT_OWNER     対象オブジェクトの所有者
OBJECT_NAME      対象オブジェクトの名前
POLICY_NAME      ポリシー名
FUNCTION         ポリシー関数
SEL              SELECT文に適用されるか
INS              INSERT文に適用されるか
UPD              UPDATE文に適用されるか
DEL              DELETE文に適用されるか
POLICY_TYPE      ポリシーのタイプ
COMMON           ポリシーの適用範囲 (YES: すべてのPDB、NO: ローカルPDBのみ)
===============  ============================================================



****************************
HRユーザーで確認
****************************

作成したVPDポリシーが正しく機能しているかを確認します。


.. code:: sql

    SQL> set markup csv on
    SQL> select employee_id, first_name, salary, job_id from hr.employees;
    "EMPLOYEE_ID","FIRST_NAME","SALARY","JOB_ID"
    100          ,"Steven"    ,24000   ,"AD_PRES"
    101          ,"Neena"     ,17000   ,"AD_VP"
    102          ,"Lex"       ,17000   ,"AD_VP"
    103          ,"Alexander" ,9000    ,"IT_PROG"
    104          ,"Bruce"     ,6000    ,"IT_PROG"
    105          ,"David"     ,4800    ,"IT_PROG"
    106          ,"Valli"     ,4800    ,"IT_PROG"
    ...
    200          ,"Jennifer"  ,4400    ,"AD_ASST"
    201          ,"Michael"   ,13000   ,"MK_MAN"
    202          ,"Pat"       ,6000    ,"MK_REP"
    203          ,"Susan"     ,6500    ,"HR_REP"
    204          ,"Hermann"   ,10000   ,"PR_REP"
    205          ,"Shelley"   ,12008   ,"AC_MGR"
    206          ,"William"   ,8300    ,"AC_ACCOUNT"

107 rows selected.


****************************
SALES_APPユーザーで確認
****************************

.. code:: sql

    sqlplus sales_app/Welcome1#Welcome1#@localhost:1521/freepdb1

    SQL> select employee_id, first_name, salary, job_id from hr.employees;
    "EMPLOYEE_ID","FIRST_NAME","SALARY","JOB_ID"
    145          ,"John"      ,0       ,"SA_MAN"
    146          ,"Karen"     ,0       ,"SA_MAN"
    147          ,"Alberto"   ,0       ,"SA_MAN"
    148          ,"Gerald"    ,0       ,"SA_MAN"
    149          ,"Eleni"     ,0       ,"SA_MAN"
    ...
    172          ,"Elizabeth" ,0       ,"SA_REP"
    173          ,"Sundita"   ,0       ,"SA_REP"
    174          ,"Ellen"     ,0       ,"SA_REP"
    175          ,"Alyssa"    ,0       ,"SA_REP"
    176          ,"Jonathon"  ,0       ,"SA_REP"
    177          ,"Jack"      ,0       ,"SA_REP"
    178          ,"Kimberely" ,0       ,"SA_REP"
    179          ,"Charles"   ,0       ,"SA_REP"

    35 rows selected.