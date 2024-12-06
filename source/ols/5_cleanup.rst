############################################
5. OLSの設定を削除する
############################################


**実施内容**

+ VPD関数の削除
+ VPDポリシーの削除


後のデモ内容と競合する可能性があるため、OLSデモで設定した関数・ポリシーを削除します。

いきなり表を削除してもいいですが、念のためこれまでと逆の手順を踏む形で進めていきます。


*************************
ポリシーの無効化
*************************
指定したポリシーをスキーマから削除します。

ポリシーはスキーマ内のすべての表から削除され、オプションでポリシーのラベル列がすべての表から削除されます。


BEGIN
    SA_POLICY_ADMIN.REMOVE_SCHEMA_POLICY(
        policy_name      => 'OLS_POL_DEMO',
        schema_name      => 'HR',
        drop_column      => TRUE);
END;
/

*************************
ユーザーラベルの削除
*************************

指定したユーザーからOracle Label Securityのすべての認可と権限を削除します。

BEGIN
    SA_USER_ADMIN.DROP_USER_ACCESS (
        policy_name   => 'OLS_POL_DEMO',
        user_name     => 'HR'); 

    SA_USER_ADMIN.DROP_USER_ACCESS (
        policy_name   => 'OLS_POL_DEMO',
        user_name     => 'SALES_APP'); 
END;
/

*************************
ラベルの削除
*************************

BEGIN
    SA_LABEL_ADMIN.DROP_LABEL (
        policy_name  => 'OLS_POL_DEMO',
        label_value  => 'SENS');

    SA_LABEL_ADMIN.DROP_LABEL (
        policy_name  => 'OLS_POL_DEMO',
        label_value  => 'CONF');

    SA_LABEL_ADMIN.DROP_LABEL (
        policy_name  => 'OLS_POL_DEMO',
        label_value  => 'INTL');
END;
/



*************************
レベルの削除
*************************

BEGIN
    SA_COMPONENTS.DROP_LEVEL (
    policy_name => 'OLS_POL_DEMO',
    short_name  => 'SENS');

    SA_COMPONENTS.DROP_LEVEL (
    policy_name => 'OLS_POL_DEMO',
    short_name  => 'CONF');

    SA_COMPONENTS.DROP_LEVEL (
    policy_name => 'OLS_POL_DEMO',
    short_name  => 'INTL');
END;
/

注意：データ・ラベルまたはユーザー・ラベルで使用されているレベルは削除できません。


*************************
ポリシーの削除
*************************

無効化するプロシージャもありますが、削除する前に無効化する必要はありませんので、いきなり削除します。

BEGIN
    SA_SYSDBA.DROP_POLICY ( 
        policy_name  => 'OLS_POL_DEMO',
        drop_column  => True);
END;
/




*************************
OLSを無効化する
*************************

Database Vaultを使用している場合は無効化しないでください。


EXEC LBACSYS.OLS_ENFORCEMENT.DISABLE_OLS;


SQL> col status for a20
SQL> col DESCRIPTION for a50
SQL> set lines 100
SQL> SELECT * FROM DBA_OLS_STATUS;

NAME                 STATUS               DESCRIPTION
-------------------- -------------------- --------------------------------------------------
OLS_CONFIGURE_STATUS TRUE                 Determines if OLS is configured
OLS_ENABLE_STATUS    FALSE                Determines if OLS is enabled


FALSEとなり、無効化されたことが分かります。

設定を完全に反映させるためにPDBの再起動を行います。

SQL> alter pluggable database freepdb1 close immediate;

Pluggable database altered.

SQL> alter pluggable database freepdb1 open;

Pluggable database altered.


以上でOracle Label Securityのデモは終了です。