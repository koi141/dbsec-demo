############################################
4. Database Vaultを無効化する
############################################

**実施内容**

+ 許可リストを作成する



********************************
レルムの無効化
********************************
BEGIN
    DBMS_MACADM.UPDATE_REALM(
        realm_name     => 'Realm for demo',
        enabled        => DBMS_MACUTL.G_NO);
END;
/


********************************
レルム認可の削除
********************************

BEGIN
    DBMS_MACADM.DELETE_AUTH_FROM_REALM(
        realm_name    => 'Realm for demo',
        grantee       => 'HR',
        auth_scope    => DBMS_MACUTL.G_SCOPE_LOCAL);
END;
/


BEGIN
    DBMS_MACADM.DELETE_AUTH_FROM_REALM(
        realm_name    => 'Realm for demo',
        grantee       => 'SALES_APP',
        auth_scope    => DBMS_MACUTL.G_SCOPE_LOCAL);
END;
/


BEGIN
    DBMS_MACADM.DELETE_AUTH_FROM_REALM(
        realm_name    => 'Realm for demo',
        grantee       => 'APP',
        auth_scope    => DBMS_MACUTL.G_SCOPE_LOCAL);
END;
/

完全に削除されたことを確認します。

    SQL> select realm_name, grantee from DVSYS.DBA_DV_REALM_AUTH where realm_name = 'Realm for demo';

    no rows selected

********************************
レルムからオブジェクトを削除する
********************************

BEGIN
    DBMS_MACADM.DELETE_OBJECT_FROM_REALM(
        realm_name   => 'Realm for demo',
        object_owner => 'HR',
        object_name  => 'COUNTRIES',
        object_type  => 'TABLE');
END;
/

BEGIN
    DBMS_MACADM.DELETE_OBJECT_FROM_REALM(
        realm_name   => 'Realm for demo',
        object_owner => 'HR',
        object_name  => 'REGIONS',
        object_type  => 'TABLE');
END;
/

オブジェクトがレルムから外されたことを確認します。

    SQL> select REALM_NAME, OWNER, OBJECT_NAME, OBJECT_TYPE from DVSYS.DBA_DV_REALM_OBJECT where realm_name = 'Realm for demo';

    no rows selected

********************************
レルムの削除
********************************
BEGIN
    DBMS_MACADM.DELETE_REALM(realm_name  => 'Realm for demo'); 
END;
/