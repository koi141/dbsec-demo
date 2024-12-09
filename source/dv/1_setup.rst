############################################
1. Database Vaultの有効化
############################################

**実施内容**

+ 許可リストを作成する


********************************
DVの構成
********************************
Oracle Database Vaultを有効化する前には、Database VaultおよびOracle Label Security (OLS) のための構築スクリプトを実行し、関連するオブジェクトやスキーマをデータベース上に作成・初期化する必要があります。
構成および有効化プロセスでは、Oracle Label Securityがまだ有効ではない場合、有効になります。Oracle Label SecurityはOracle Database Vaultに必要ですが、別にOracle Label Securityの使用を開始してOracle Label Securityポリシーを作成する場合を除き、別個のライセンスは必要ありません。

関連するPDBで同じアクションを実行する前に、CDBルートでOracle Database Vaultを構成して有効化する必要があります。

CDBルートでDV_OWNERロールとDV_ACCTMGRロールを割り当てられた共通ユーザーも、PDBで同じロールを持つことができます。
PDBでは、同じ共通ユーザーを使用してDatabase Vaultを構成および有効化することも、別のPDBローカル・ユーザーを使用することもできます。DV_ACCTMGRロールは、CDBルートの共通ユーザーに共通に付与されます。
Database VaultをCDBルートに構成および有効化するときに、DV_OWNERをローカルに、またはCDBルート共通ユーザーに共通に付与できます。
DV_OWNERを共通ユーザーにローカルに付与すると、共通DV_OWNERユーザーは、どのPDBでもこのロールを使用できなくなります。

今回はDV管理者としては共通ユーザを使用することとします。

CDB$ROOTに接続し、オプションの設定を確認します。

    SQL> SELECT * FROM CDB_DV_STATUS;

    NAME                STATUS             CON_ID
    ------------------- -------------- ----------
    DV_CONFIGURE_STATUS FALSE                   1
    DV_ENABLE_STATUS    FALSE                   1
    DV_APP_PROTECTION   NOT CONFIGURED          1
    DV_CONFIGURE_STATUS FALSE                   3
    DV_ENABLE_STATUS    FALSE                   3
    DV_APP_PROTECTION   NOT CONFIGURED          3

    6 rows selected.

********************************
(CDB$ROOT) DV管理者ユーザーを作成する
********************************
DV管理ユーザーを作成します。
登録プロセス中に、Oracle Database Vault 所有者と Oracle Database Vault アカウント マネージャのアカウントを作成する必要があります。

インストール中に、Oracle Database Vault 所有者アカウントのアカウント名とパスワードを指定する必要があります。Oracle Database Vault アカウント マネージャの作成はオプションですが、職務の分離を改善するために強く推奨されます。

Oracle Database Vault 所有者アカウントにDV_OWNERロールが付与されます。このアカウントは、Oracle Database Vault のロールと構成を管理できます。

Oracle Database Vault アカウント マネージャ アカウントにDV_ACCTMGRロールが付与されます。このアカウントは、職務の分離を容易にするためにデータベース ユーザー アカウントを管理するために使用されます。

インストール中に Oracle Database Vault アカウント マネージャ アカウントを作成しないことを選択した場合は、DV_OWNERと の両方のDV_ACCTMGRロールが Oracle Database Vault 所有者ユーザー アカウントに付与されます。

ここでは以下の2つのユーザーを作成します。ロール名と名前が似ていますので注意してください。

CDBの共通ユーザとして作成することも可能ですが、ここではローカルユーザーを作成していきます。


共通ユーザの接頭辞を確認します。

    SQL> set markup csv on
    SQL> select name, value from v$parameter where name = 'common_user_prefix';
    "NAME","VALUE"
    "common_user_prefix","C##"

Database Vault所有者(DV_OWNERロール)およびDatabase Vaultアカウント・マネージャ(DV_ACCTMGRロール)のアカウント用に使用されるユーザー・アカウントを選択(または新規ユーザーを作成)選択します。
ロールごとに2つのアカウントを維持することをお薦めします。一方のアカウントはその名前付きユーザーのプライマリ・アカウントであり、日常的に使用されます。他方のアカウントは、プライマリ・アカウントのパスワードを忘れてしまいリセットする必要がある場合に備えたバックアップ・アカウントとして使用されます。
アカウントのバックアップがない場合は、パスワードをリセットできません。

**C##DVOWNER**
    セキュリティ管理者
    ロール: DV_OWNER

**C##DVACCTMGR**
    アカウント管理者    
    アカウントを作成する
        ロール: DV_ACCMGR

GRANT CREATE SESSION TO DVOWNER IDENTIFIED BY Welcome1#Welcome1#;

grant create session, set container to C##DVOWNER identified by Welcome1#Welcome1# container = all;
grant create session, set container to C##DVACCTMGR identified by Welcome1#Welcome1# container = all;

********************************
(CDB$ROOT) DVの構成
********************************
作成した2つのユーザーを指定して、DVを構成します。

BEGIN
    CONFIGURE_DV (
        dvowner_uname        => 'C##DVOWNER',    
        dvacctmgr_uname      => 'C##DVACCTMGR',
        force_local_dvowner  => FALSE);
END;
/

この例では、force_local_dvownerをFALSEに設定すると、共通ユーザーは、このCDBルートに関連付けられているPDBのDV_OWNER権限を持つことができます。
TRUEに設定すると、共通DV_OWNERユーザーはCDBルートにのみDV_OWNERロール権限を持つように制限されます。

構成ステータスがTrueになっていることが分かります

    SQL> SELECT * FROM CDB_DV_STATUS;
    "NAME","STATUS","CON_ID"
    "DV_CONFIGURE_STATUS","TRUE",1
    "DV_ENABLE_STATUS","FALSE",1
    "DV_APP_PROTECTION","NOT CONFIGURED",1
    "DV_CONFIGURE_STATUS","FALSE",3
    "DV_ENABLE_STATUS","FALSE",3
    "DV_APP_PROTECTION","NOT CONFIGURED",3

    6 rows selected.

utlrp.sqlスクリプトを実行し、無効化状態となっているオブジェクトをコンパイルします。

    SQL> @?/rdbms/admin/utlrp.sql
    ...
    "OBJECTS WITH ERRORS"
    0
    ...
    "ERRORS DURING RECOMPILATION"
    0
    ...

問題なく、実行が完了することを確認します。


********************************
(CDB$ROOT) DVを有効化する
********************************
Oracle Database Vault (DB Vault)をマルチテナント環境(CDB)で有効化する際には、大きく分けて「通常(非厳密)モード」と「厳密モード」の2つの動作モードを選択できます。
これらのモードは、CDB全体でDB Vaultが有効化されている際に、PDB(Pluggable Database)ごとのDB Vault有効化状態がどのように扱われるかを制御します。

+ 通常モード
    CDBでDB Vaultが有効化されている場合でも、PDB単位でDB Vaultが有効化されているかどうかにかかわらず、そのPDBは通常通り機能を継続します。
    つまり、CDBではDB Vaultが有効であっても、PDBレベルで無効な状態のままでもPDBは使い続けることができます。

+ 厳密モード
    厳密モードでは、CDBがDB Vault有効状態になった時点で、PDBを読み書きモードでオープンするにはそのPDBでもDB Vaultが構成・有効化されている必要があります。
    簡単に言えば、「CDBでDB Vaultが有効なら、すべてのPDBもDB Vaultを有効化していないと開けない」という制限が課されます。

今回はPDBだけDVを有効化していきますので、「通常モード」で有効化していきます。

先ほど作成し、DV管理者として指定したC##DVOWNERでCDBに接続します。

    $ sqlplus C##DVOWNER/Welcome1#Welcome1#

    SQL> show user con_name
    USER is "C##DVOWNER"

    CON_NAME
    ------------------------------
    CDB$ROOT

通常モードで有効化します。

    SQL> EXEC DBMS_MACADM.ENABLE_DV;

    PL/SQL procedure successfully completed.

有効化ステータスがTRUEになっていることが確認できます

    SQL> SELECT * FROM CDB_DV_STATUS;
    "NAME","STATUS","CON_ID"
    "DV_CONFIGURE_STATUS","TRUE",1
    "DV_ENABLE_STATUS","TRUE",1
    "DV_APP_PROTECTION","NOT CONFIGURED",1

SYSユーザーで再び接続し、DBを再起動します。

    SQL> shutdown immediate
    SQL> startup
    -- PDBがオープンされていることを確認
    SQL> show pdbs;

        CON_ID CON_NAME                       OPEN MODE  RESTRICTED
    ---------- ------------------------------ ---------- ----------
            2 PDB$SEED                       READ ONLY  NO
            3 FREEPDB1                       READ WRITE NO


再起動後、DVとOLSが有効化されていることを最後に確認します

    SQL> set markup csv on
    SQL> SELECT parameter, VALUE FROM V$OPTION WHERE PARAMETER IN ('Oracle Database Vault','Oracle Label Security');
    "PARAMETER","VALUE"
    "Oracle Label Security","TRUE"
    "Oracle Database Vault","TRUE"


********************************
(PDB) 設定の確認
********************************
PDBに接続し、改めてオプションの有効・無効を確認します。
DVでは、OLSも必要になってきますので、同時に確認しています。
同時に有効化されるため、OLSの有効化設定は必要ありません。

SQL> set markup csv on
SQL> SELECT parameter, VALUE FROM V$OPTION WHERE PARAMETER IN ('Oracle Database Vault','Oracle Label Security');
"PARAMETER","VALUE"
"Oracle Label Security","FALSE"
"Oracle Database Vault","FALSE"

PDBのdatabase Vaultのステータスを確認します。

SQL> SELECT * FROM DBA_DV_STATUS;
"NAME","STATUS"
"DV_CONFIGURE_STATUS","FALSE"
"DV_ENABLE_STATUS","FALSE"
"DV_APP_PROTECTION","NOT CONFIGURED"



FREEPDB1に接続し、DVを構成します。SYSユーザーで大丈夫です。

BEGIN
    CONFIGURE_DV (
        dvowner_uname        => 'C##DVOWNER',    
        dvacctmgr_uname      => 'C##DVACCTMGR');
END;
/

SQL> SELECT * FROM DBA_DV_STATUS;
"NAME","STATUS"
"DV_CONFIGURE_STATUS","TRUE"
"DV_ENABLE_STATUS","FALSE"
"DV_APP_PROTECTION","NOT CONFIGURED"

構成がTRUEになったことが確認できます。


utlrp.sqlスクリプトを実行し、無効化状態となっているオブジェクトをコンパイルします。

    SQL> @?/rdbms/admin/utlrp.sql
    ...
    "OBJECTS WITH ERRORS"
    0
    ...
    "ERRORS DURING RECOMPILATION"
    0
    ...

問題なく、実行が完了することを確認します。



********************************
(PDB) DVを有効化する
********************************
先ほど構成したバックアップDatabase Vault所有者ユーザーとして、PDBに接続します。

    $ sqlplus c##dvowner/Welcome1#Welcome1#@localhost:1521/FREEPDB1

    SQL> show user con_name
    USER is "C##DVOWNER"

    CON_NAME
    ------------------------------
    FREEPDB1

DVを有効化します

    SQL> EXEC DBMS_MACADM.ENABLE_DV;

    PL/SQL procedure successfully completed.

CDBにSYSユーザーで接続し、PDBを再起動します。

    SQL> conn / as sysdba
    Connected.
    SQL> show user con_name
    USER is "SYS"

    CON_NAME
    ------------------------------
    CDB$ROOT

    SQL> alter pluggable database freepdb1 close immediate;

    SQL> alter pluggable database freepdb1 open;

DVおよびOLSが有効化されたことを確認します

    SQL> SELECT * FROM DBA_DV_STATUS;

    NAME                STATUS
    ------------------- --------------
    DV_CONFIGURE_STATUS TRUE
    DV_ENABLE_STATUS    TRUE
    DV_APP_PROTECTION   NOT CONFIGURED

    SQL> col DESCRIPTION for a40
    SQL> SELECT * FROM DBA_OLS_STATUS;

    NAME                 STATU DESCRIPTION
    -------------------- ----- ----------------------------------------
    OLS_CONFIGURE_STATUS TRUE  Determines if OLS is configured
    OLS_ENABLE_STATUS    TRUE  Determines if OLS is enabled

