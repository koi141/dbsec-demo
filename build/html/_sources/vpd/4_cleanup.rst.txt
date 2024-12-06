############################################
3. クライアント識別子を用いた制御
############################################


**実施内容**

+ VPD関数の削除
+ VPDポリシーの削除


後のデモ内容と競合する可能性があるため、VPDデモで作成した関数・ポリシーを削除します。

*************************
VPD関数の削除
*************************

SQL> select object_name from all_objects where owner = 'HR' and object_type = 'FUNCTION';

OBJECT_NAME
--------------------------------------------------------------------------------
GET_SALES_PREDICATE
GET_MASKING_SALARY_COL

drop function hr.get_sales_predicate;


SQL> drop function hr.get_sales_predicate;

Function dropped.


SQL> drop function hr.get_masking_salary_col;

Function dropped.

削除されたかを確認します

SQL> select object_name from all_objects where owner = 'HR' and object_type = 'FUNCTION';

no rows selected



*************************
VPDポリシーの削除
*************************

ポリシーを確認します。
SQL> select POLICY_NAME from all_policies where object_owner  = 'HR';

POLICY_NAME
--------------------------------------------------------------------------------
EMPLOYEES_SALARY_COL_VPD_POLICY
EMPLOYEES_VPD_POLICY



行制御ポリシー
=================

BEGIN
    DBMS_RLS.DROP_POLICY(
        object_schema => 'HR',
        object_name   => 'EMPLOYEES',
        policy_name   => 'employees_vpd_policy'
    );
END;
/


列制御ポリシー
=================

BEGIN
    DBMS_RLS.DROP_POLICY(
        object_schema => 'HR',
        object_name   => 'EMPLOYEES',
        policy_name   => 'employees_salary_col_vpd_policy'
    );
END;
/

削除されたかを確認します。

SQL> select POLICY_NAME from all_policies where object_owner  = 'HR';

no rows selected

