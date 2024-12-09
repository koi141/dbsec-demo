############################################
2. Database Vaultの準備
############################################

**実施内容**

+ 許可リストを作成する



********************************
確認
********************************

有効化した時点でSYSユーザーではユーザーが作成できないことが確認できます。

    SQL> show user
    USER is "SYS"
    SQL> create user test;
    create user test
    *
    ERROR at line 1:
    ORA-01031: insufficient privileges
    Help: https://docs.oracle.com/error-help/db/ora-01031/

レルムはまだ作成していないので、表を参照することはできます。

    SQL> select count(*) from hr.jobs;

    COUNT(*)
    ----------
            19

********************************
レルムを作成する
********************************

他の機能と競合しないよう、念のため今回はHRスキーマのREGIONS表とCOUNTRIES表をレルムに追加します。

DV管理者で以下を実行します。


BEGIN
    DBMS_MACADM.CREATE_REALM(
        realm_name        => 'Realm for demo',              -- レルム名
        description       => 'This realm is created for demonstration',  -- レルムの説明
        enabled           => DBMS_MACUTL.G_YES,             -- (*)作成直後から有効化
	    audit_options     => DBMS_MACUTL.G_REALM_AUDIT_OFF, -- (*)レルムの監査を無効
	    realm_type        => 1,                             -- 必須レルムを有効化
	    realm_scope       => DBMS_MACUTL.G_SCOPE_LOCAL,     -- (*)レルムはローカルの範囲で動作
	    pl_sql_stack      => FALSE                          -- (*)PL/SQLスタック検証は行わない
    );
END;
/

作成したレルムは ``DVSYS.DBA_DV_REALM`` から確認することができます。

    SQL> select name, description, realm_type from dvsys.dba_dv_realm;
    "NAME","DESCRIPTION","REALM_TYPE"
    "Oracle Database Vault","Defines the realm for the Oracle Database Vault schemas - DVSYS and DVF where Database Vault access control configuration and roles are contained.","MANDATORY"
    "Oracle Label Security","Defines the realm for the Oracle Label Security schemas and roles - LBACSYS and LBAC_DBA.","MANDATORY"
    "Database Vault Account Management","Defines the realm for administrators who create and manage database accounts and profiles.","REGULAR"
    "Oracle Enterprise Manager","Defines the Enterprise Manager monitoring and management realm.","REGULAR"
    "Oracle Default Schema Protection Realm","Defines the realm for the Oracle Default schemas.","REGULAR"
    "Oracle System Privilege and Role Management Realm","Defines the realm to control granting of system privileges and database administrator roles.","REGULAR"
    "Oracle Default Component Protection Realm","Defines the realm to protect default components of the Oracle database.","REGULAR"
    "Oracle Audit","Defines the realm to protect audit related objects of the Oracle database.","MANDATORY"
    "Oracle GoldenGate Protection Realm","Defines the realm to protect GoldenGate-related objects of the Oracle database.","MANDATORY"
    "Realm for demo","This realm is created for demonstration","MANDATORY"

    10 rows selected.


********************************
レルムにオブジェクトを登録する
********************************

レルム認可を必要とするオブジェクトを登録します。

BEGIN
    DBMS_MACADM.ADD_OBJECT_TO_REALM(
        realm_name        => 'Realm for demo',
        object_owner      => 'HR',
        object_name       => 'COUNTRIES',
        object_type       => 'TABLE'
    );
END;
/

BEGIN
    DBMS_MACADM.ADD_OBJECT_TO_REALM(
        realm_name        => 'Realm for demo',
        object_owner      => 'HR',
        object_name       => 'REGIONS',
        object_type       => 'TABLE'
    );
END;
/

object_name, object_typeではワイルドカード'%'が使用することができますので、HRスキーマ内のオブジェクトを一括で登録することが可能です。


登録したオブジェクトは以下のコマンドで確認できます。

    SQL> select REALM_NAME, OWNER, OBJECT_NAME, OBJECT_TYPE from DVSYS.DBA_DV_REALM_OBJECT where realm_name = 'Realm for demo';
    "REALM_NAME","OWNER","OBJECT_NAME","OBJECT_TYPE"
    "Realm for demo","HR","COUNTRIES","TABLE"
    "Realm for demo","HR","REGIONS","TABLE"


********************************
レルム認可を行う
********************************
このままではオブジェクトの持ち主であるHRユーザーでさえも、レルム内のオブジェクトにアクセスすることができません。
そのため、レルム認可を行い、オブジェクトにアクセスする権限を付与します。

所有者(HR)
=======================

BEGIN
	DBMS_MACADM.ADD_AUTH_TO_REALM(
		realm_name        => 'Realm for demo',   -- レルム名
		grantee           => 'HR',               -- 権限を付与するユーザ名またはロール名
		auth_options      => DBMS_MACUTL.G_REALM_AUTH_OWNER  -- ユーザーを「所有者」として認可する
    );
END;
/

参加者(SALES_APPユーザー)
==========================

BEGIN
	DBMS_MACADM.ADD_AUTH_TO_REALM(
		realm_name        => 'Realm for demo',   -- レルム名
		grantee           => 'SALES_APP',           -- 権限を付与するユーザ名またはロール名
		auth_options      => DBMS_MACUTL.G_REALM_AUTH_PARTICIPANT  -- ユーザーを「参加者」として認可する
    );
END;
/


参加者(APPユーザー)
==========================
APPユーザーに対してはIPアドレスでの制限を追加してみます。

ルールを作成します
BEGIN
    DBMS_MACADM.CREATE_RULE(
	    rule_name       => 'Rule to restrict APP to specific IP', 
        rule_expr       => 'SYS_CONTEXT(''USERENV'',''IP_ADDRESS'') = ''192.168.0.10''',
        scope           => DBMS_MACUTL.G_SCOPE_LOCAL
    );
END;
/

ルールを束ねたルールセットを作成します。
BEGIN
	DBMS_MACADM.CREATE_RULE_SET(
		rule_set_name    => 'Ruleset for APP', 
		description      => 'Rule to restrict APP to specific IP', 
		enabled          => DBMS_MACUTL.G_YES,   -- (*)
		eval_options     => DBMS_MACUTL.G_RULESET_EVAL_ALL,    -- (*)
		audit_options    => DBMS_MACUTL.G_RULESET_AUDIT_OFF,   -- (*)
		fail_options     => DBMS_MACUTL.G_RULESET_FAIL_SHOW,   -- (*)
		fail_message     => 'DV_Error: Can only be accessed from a specific IP address', 
		fail_code        => 20000, 
		handler_options  => DBMS_MACUTL.G_RULESET_HANDLER_OFF,   -- (*)
		handler          => '',
		is_static        => FALSE, -- (*)
		scope            => DBMS_MACUTL.G_SCOPE_LOCAL
	);
END;
/

ルールセットにルールを追加します。

BEGIN
	DBMS_MACADM.ADD_RULE_TO_RULE_SET(
		rule_set_name  => 'Ruleset for APP', 
		rule_name      => 'Rule to restrict APP to specific IP', 
		rule_order     => 1, 
		enabled        => DBMS_MACUTL.G_YES -- (*)
	);
END;
/

BEGIN
	DBMS_MACADM.ADD_AUTH_TO_REALM(
		realm_name        => 'Realm for demo',   -- レルム名
		grantee           => 'APP',           -- 権限を付与するユーザ名またはロール名
        rule_set_name     => 'Ruleset for APP',
		auth_options      => DBMS_MACUTL.G_REALM_AUTH_PARTICIPANT  -- ユーザーを「参加者」として認可する
    );
END;
/

レルム認可は以下のコマンドで見ることができます。
SQL> select realm_name, grantee, AUTH_OPTIONS,AUTH_RULE_SET_NAME from DVSYS.DBA_DV_REALM_AUTH where realm_name = 'Realm for demo';
"REALM_NAME","GRANTEE","AUTH_OPTIONS","AUTH_RULE_SET_NAME"
"Realm for demo","APP","Participant","Ruleset for APP"
"Realm for demo","SALES_APP","Participant",
"Realm for demo","HR","Owner",
