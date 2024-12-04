###########################
2. VPDで列制御を行う
###########################

**実施内容**

+ VPD関数の作成
+ VPDのポリシーの作成
+ HRユーザーまたはSYSユーザーでEMMPLOYEES表を確認
+ SALES_APPユーザーでEMMPLOYEES表を確認


where句を使用すると行レベルでの制御になりますが、VPDでは列を制御することも可能です。
この手順ではVPDを用いて列を制御してみます。

SALES_APPユーザーには ``salary`` 列を見せる必要がないと想定して、設定を行っていきます。


****************************
VPD関数の作成
****************************

.. code:: sql

CREATE OR REPLACE FUNCTION hr.get_masking_salary_col( 
    p_schema IN VARCHAR2,
    p_table  IN VARCHAR2
    )
    RETURN VARCHAR2
    IS
        v_predicate VARCHAR2 (400);
    BEGIN
        IF SYS_CONTEXT('USERENV', 'SESSION_USER') = 'SALES_APP' THEN
        v_predicate := '1=2';
    END IF;
    RETURN v_predicate;
END get_masking_salary_col;
/


実行し、 ``Function created.`` が表示されることを確認します。

v_predicate が必ずfalseになるように設定します。


****************************
VPDポリシーの作成
****************************


BEGIN
    DBMS_RLS.ADD_POLICY (
        object_schema         => 'HR',
        object_name           => 'EMPLOYEES',
        policy_name           => 'employees_salary_col_vpd_policy',
        function_schema       => 'HR',
        policy_function       => 'get_masking_salary_col',
        sec_relevant_cols     => 'SALARY'
    );
END;
/

実行し、 ``PL/SQL procedure successfully completed.`` が表示されることを確認します。

sec_relevant_cols_opt => dbms_rls.ALL_ROWS
を選択すると、結果の値は返さないもののsalary列自体は返されるようになります。
デフォルトではNULLとなっていますので、上記コマンドでは指定していません。

作成したVPDポリシーを確認します。



SQL> set markup csv on
SQL> select object_owner, object_name, policy_name, function, sel, ins, upd, del, idx, policy_type, common from all_policies where object_owner  = 'HR';
"OBJECT_OWNER","OBJECT_NAME","POLICY_NAME"                    ,"FUNCTION"              ,"SEL","INS","UPD","DEL","IDX","POLICY_TYPE","COMMON"
"HR"          ,"EMPLOYEES"  ,"EMPLOYEES_VPD_POLICY"           ,"GET_SALES_PREDICATE"   ,"YES","NO" ,"YES","YES","NO" ,"DYNAMIC"    ,"NO"
"HR"          ,"EMPLOYEES"  ,"EMPLOYEES_SALARY_COL_VPD_POLICY","GET_MASKING_SALARY_COL","YES","NO" ,"NO" ,"NO" ,"NO" ,"DYNAMIC"    ,"NO"

前手順で作成した ``EMPLOYEES_VPD_POLICY`` に加えてポリシーが作成されたことを確認します。


****************************
HRユーザーで確認
****************************

作成したVPDポリシーが正しく機能しているかを確認します。


SQL> desc hr.employees
 Name                                      Null?    Type
 ----------------------------------------- -------- ----------------------------
 EMPLOYEE_ID                               NOT NULL NUMBER(6)
 FIRST_NAME                                         VARCHAR2(20)
 LAST_NAME                                 NOT NULL VARCHAR2(25)
 EMAIL                                     NOT NULL VARCHAR2(25)
 PHONE_NUMBER                                       VARCHAR2(20)
 HIRE_DATE                                 NOT NULL DATE
 JOB_ID                                    NOT NULL VARCHAR2(10)
 SALARY                                             NUMBER(8,2)
 COMMISSION_PCT                                     NUMBER(2,2)
 MANAGER_ID                                         NUMBER(6)
 DEPARTMENT_ID                                      NUMBER(4)


SQL> select first_name, salary from hr.employees;
"FIRST_NAME","SALARY"
"Steven",24000
"Neena",17000
"Lex",17000
"Alexander",9000
"Bruce",6000
"David",4800
"Valli",4800
"Diana",4200
...
"Donald",2600
"Douglas",2600
"Jennifer",4400
"Michael",13000
"Pat",6000
"Susan",6500
"Hermann",10000
"Shelley",12008
"William",8300

107 rows selected.


SQL> select * from hr.employees;
"EMPLOYEE_ID","FIRST_NAME","LAST_NAME","EMAIL","PHONE_NUMBER","HIRE_DATE","JOB_ID","SALARY","COMMISSION_PCT","MANAGER_ID","DEPARTMENT_ID"
100,"Steven","King","SKING","1.515.555.0100","17-JUN-13","AD_PRES",24000,,,90
101,"Neena","Yang","NYANG","1.515.555.0101","21-SEP-15","AD_VP",17000,,100,90
...


****************************************************************************
SALES_APPユーザーで確認
****************************************************************************

SQL> desc hr.employees;
 Name                                      Null?    Type
 ----------------------------------------- -------- ----------------------------
 EMPLOYEE_ID                               NOT NULL NUMBER(6)
 FIRST_NAME                                         VARCHAR2(20)
 LAST_NAME                                 NOT NULL VARCHAR2(25)
 EMAIL                                     NOT NULL VARCHAR2(25)
 PHONE_NUMBER                                       VARCHAR2(20)
 HIRE_DATE                                 NOT NULL DATE
 JOB_ID                                    NOT NULL VARCHAR2(10)
 SALARY                                             NUMBER(8,2)
 COMMISSION_PCT                                     NUMBER(2,2)
 MANAGER_ID                                         NUMBER(6)
 DEPARTMENT_ID                                      NUMBER(4)

SQL> select first_name, salary from hr.employees;

no rows selected

SQL> select * from hr.employees;

no rows selected


SQL> select first_name from hr.employees;
"FIRST_NAME"
"Ellen"
"Sundar"
"Amit"
"Elizabeth"
"David"
"Harrison"
"Gerald"
"Nanette"
"Louise"
...
"John"
"Lindsey"
"William"
"Patrick"
"Jonathon"
"Sean"
"Oliver"
"Clara"
"Eleni"

35 rows selected.

salary列が選択されたときにVPDポリシーが発動します。


****************************************************************************
SALES_APPユーザーで確認（sec_relevant_cols_opt => dbms_rls.ALL_ROWSとした場合）
****************************************************************************

ポリシーを再作成する際は、以下を実行して削除してから、また作成します。

BEGIN
    DBMS_RLS.DROP_POLICY(
        object_schema => 'HR',
        object_name   => 'EMPLOYEES',
        policy_name   => 'employees_salary_col_vpd_policy'
    );
END;
/




SQL> desc hr.employees
 Name                                      Null?    Type
 ----------------------------------------- -------- ----------------------------
 EMPLOYEE_ID                               NOT NULL NUMBER(6)
 FIRST_NAME                                         VARCHAR2(20)
 LAST_NAME                                 NOT NULL VARCHAR2(25)
 EMAIL                                     NOT NULL VARCHAR2(25)
 PHONE_NUMBER                                       VARCHAR2(20)
 HIRE_DATE                                 NOT NULL DATE
 JOB_ID                                    NOT NULL VARCHAR2(10)
 SALARY                                             NUMBER(8,2)
 COMMISSION_PCT                                     NUMBER(2,2)
 MANAGER_ID                                         NUMBER(6)
 DEPARTMENT_ID                                      NUMBER(4)


SQL> select first_name, salary from hr.employees;
"FIRST_NAME","SALARY"
"John",
"Karen",
"Alberto",
"Gerald",
"Eleni",
"Sean",
"David",
"Peter",
"Christopher",
"Nanette",
...
"Tayler",
"William",
"Elizabeth",
"Sundita",
"Ellen",
"Alyssa",
"Jonathon",
"Jack",
"Kimberely",
"Charles",

35 rows selected.


SQL> select * from hr.employees;
"EMPLOYEE_ID","FIRST_NAME","LAST_NAME","EMAIL","PHONE_NUMBER","HIRE_DATE","JOB_ID","SALARY","COMMISSION_PCT","MANAGER_ID","DEPARTMENT_ID"
145,"John","Singh","JSINGH","44.1632.960000","01-OCT-14","SA_MAN",,0,100,80
146,"Karen","Partners","KPARTNER","44.1632.960001","05-JAN-15","SA_MAN",,0,100,80
...


このようにsalary列の存在が確認できます。