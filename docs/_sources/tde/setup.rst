###########################
TDE セットアップ
###########################

****************************
初期化パラメータの確認
****************************

TDEに関連する初期化パラメータは2つあります。

+ `wallet_root <https://docs.oracle.com/cd/F82042_01/refrn/WALLET_ROOT.html>`__ : 暗号化キー（TDEキー）や証明書などを保存する場所を指定
+ `tde_configuration <https://docs.oracle.com/cd/F82042_01/refrn/TDE_CONFIGURATION.html>`__ : TDEの設定を定義

それぞれを確認していきましょう。


wallet_root
============================


pdbに接続し現在の値を確認します。（見やすくなるよう、結果を整形しています）

.. code:: sql

   -- 結果をcsv形式で出力
   SQL> set markup csv on

   -- wallet_rootの値を確認
   SQL> select inst_id, name, value, issys_modifiable from gv$parameter where name = 'wallet_root';
   "INST_ID","NAME"       ,"VALUE","ISSYS_MODIFIABLE"
   1        ,"wallet_root",       ,"FALSE"

デフォルト値はないため、何も設定されていないことが確認できます。  
列の1つ ``ISSYS_MODIFIABLE`` は ``ALTER SYSTEM`` コマンドでパラメータを変更できるかを表しているため、 ``FALSE`` は設定の反映に再起動が必要なことを表しています。（つまり静的パラメータということです）


tde_configuration
============================

同様に現在の値を確認します。

.. code:: sql

   SQL> select inst_id, name, value, issys_modifiable from gv$parameter where name = 'tde_configuration';
   "INST_ID","NAME"             ,"VALUE","ISSYS_MODIFIABLE"
   1        ,"tde_configuration",       ,"IMMEDIATE"

こちらもデフォルト値はないため、何も設定されていないことが確認できます。
``ISSYS_MODIFIABLE`` は ``IMMEDIATE`` ですので、即時反映される動的パラメータであることが分かります。



****************************
初期化パラメータの設定
****************************

ではそれぞれのパラメータを設定していきます。

wallet_root
============================

TDEの暗号化キーはOracle Walletに保存されます。
wallet_rootでこのWalletを格納するディレクトリのパスを指定します。

ディレクトリはどこでもよいのですが、ここでは ``/opt/oracle/admin/FREE/wallet`` を指定することにします。
walletディレクトリは作成されていませんので、事前にそのディレクトリを準備しておきます。

.. code:: bash

   $ mkdir -p /opt/oracle/admin/FREE/wallet


CDBに接続し以下のコマンドでパラメータを設定します。PDBからの実行は許されていません。

.. code:: sql

   SQL> alter system set wallet_root='/opt/oracle/admin/FREE/wallet' scope=spfile;

   System altered.

   -- 静的パラメータなので再起動するまで反映されていないことが分かる
   SQL> select inst_id, name, value, issys_modifiable from gv$parameter where name = 'wallet_root';
   "INST_ID","NAME"       ,"VALUE","ISSYS_MODIFIABLE"
   1        ,"wallet_root",       ,"FALSE"

   -- 再起動する
   SQL> shutdown immediate
   SQL> startup

   -- 再起動後、設定が反映されている
   SQL> select inst_id, name, value, issys_modifiable from gv$parameter where name = 'wallet_root';
   "INST_ID","NAME"       ,"VALUE"                        ,"ISSYS_MODIFIABLE"
   1        ,"wallet_root","/opt/oracle/admin/FREE/wallet","FALSE"


tde_configuration
============================

tde_configurationはTDEで使用されるキーストアの種類を設定します。
19cより分離モードがサポートされ、PDBごとに固有のキーストアを使用することができるようになりました。

| サポートされるキーストアは以下の通りです。
| （参考： https://docs.oracle.com/cd/F82042_01/asoag/introduction-to-transparent-data-encryption.html ）

.. image:: ../_static/tde/サポートされるキーストア.png

なお、設定のためにはwallet_rootを有効にしておく必要があります。

有効化するとwallet_root配下にディレクトリが作成されます。

:FILE: ``<WALLET_ROOT>/tde``
:Oracle Key Vault: ``<WALLET_ROOT>/okv``

今回はデモですので、HSMなどの外部キーストアは使用せず、DBサーバーにキーストアを設置します。

.. code:: sql

   SQL> alter system set tde_configuration='keystore_configuration=file' scope=both;

   -- すぐに反映されていることが確認できる
   SQL> select inst_id, name , value , issys_modifiable from gv$parameter where name = 'tde_configuration';
   "INST_ID","NAME"             ,"VALUE"                      ,"ISSYS_MODIFIABLE"
   1        ,"tde_configuration","keystore_configuration=file","IMMEDIATE"

CDBで設定を行った場合、PDBはCDBから値を継承します。



****************************
キーストアの作成
****************************

暗号化鍵を格納するためのキーストアを作成します。
キーストアのマスター鍵管理はSYSKM権限以上が必要です。

こちらのキーストア操作はSYSユーザーでも可能ですが、キーストア操作の専用ユーザーとしてsyskmユーザーが用意されていますので、こちらを使用しても構いません。

.. code-block:: sql
   :caption: CDBで実行 (syskmユーザー)

   # oracleユーザーでSYSKMとして接続
   $ sqlplus / as syskm

   SQL> show user
   USER is "SYSKM"

   -- SYSKMユーザーのもつ権限を確認
   SQL> select * from session_privs;

   PRIVILEGE
   ----------------------------------------
   SYSKM
   ADMINISTER KEY MANAGEMENT

キーストアを作成します。デフォルトではPKCS#12ベースのキーストレージファイルに保存されます。
https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/ADMINISTER-KEY-MANAGEMENT.html

CDBから以下のコマンドを実行します。

.. code-block:: sql
   :caption: CDBで実行 (syskmユーザー)

   SQL> administer key management create keystore identified by OracleKM123#;

   keystore altered.

このコマンドを実行すると ``<wallet_root>/tde`` ディレクトリが作成され、その中に ``ewallet.p12`` が作成されます。

.. code:: bash

   $ pwd && tree
   /opt/oracle/admin/FREE/wallet
   .
   └── tde
      └── ewallet.p12

``V$ENCRYPTION_WALLETビュー`` からも確認することができます。

.. code-block:: sql
   :caption: CDBで実行 (syskmユーザー)

   SQL> set markup csv on
   SQL> select * from v$encryption_wallet;
   "WRL_TYPE","WRL_PARAMETER"                     ,"STATUS","WALLET_TYPE","WALLET_ORDER","KEYSTORE_MODE","FULLY_BACKED_UP","CON_ID"
   "FILE"    ,"/opt/oracle/admin/FREE/wallet/tde/","CLOSED","UNKNOWN"    ,"SINGLE"      ,"NONE"         ,"UNDEFINED"      ,1

   -- キーストアがCLOSEDになっていますので、OPENにします。
   SQL> administer key management set keystore open identified by OracleKM123#;
   
   -- STATUS列がOPENになっていることを確認
   SQL> select * from v$encryption_wallet;
   "WRL_TYPE","WRL_PARAMETER","STATUS","WALLET_TYPE","WALLET_ORDER","KEYSTORE_MODE","FULLY_BACKED_UP","CON_ID"
   "FILE","/opt/oracle/admin/FREE/wallet/tde/","OPEN_NO_MASTER_KEY","PASSWORD","SINGLE","NONE","UNDEFINED",1


****************************
マスター暗号鍵の作成
****************************

続いてマスター暗号鍵を作成します。今回は統合モードでCDB、PDBを一括で含めた暗号化鍵を作ることにします。

.. code-block:: sql
   :caption: CDBで実行 (syskmユーザーまたはsysユーザー)

   SQL> administer key management set key using tag 'v1.0_MEK_AllContainer' identified by OracleKM123# with backup container = ALL;

   keystore altered.

``using tag`` 句は無くても問題ないですが、管理のために付けておくとよいかと思います。

.. code-block:: sql
   :caption: PDBで実行 (sysユーザー)

   -- PDBから正しくウォレットを認識できているかを確認
   SQL> select * from v$encryption_wallet;
   "WRL_TYPE","WRL_PARAMETER","STATUS","WALLET_TYPE","WALLET_ORDER","KEYSTORE_MODE","FULLY_BACKED_UP","CON_ID"
   "FILE"    ,               ,"OPEN"  ,"PASSWORD"   ,"SINGLE"      ,"UNITED"       ,"NO"             ,3

   -- PDBから正しくマスター暗号鍵を認識できているかを確認（都合上KEY_IDは短縮しています）
   SQL> select key_id, tag, creator, user, key_use, keystore_type, activating_dbname from v$encryption_keys;
   "KEY_ID"      ,"TAG"                  , "CREATOR","USER","KEY_USE"   ,"KEYSTORE_TYPE"    ,"ACTIVATING_DBNAME"
   "AU1kv...AAAA","v1.0_MEK_AllContainer", "SYSKM"  ,"SYS" ,"TDE IN PDB","SOFTWARE KEYSTORE","FREE"


:CREATOR: マスター・キーを作成したユーザー
:USER: マスター・キーをアクティブ化したユーザー

では、次の手順から実際に表領域を暗号化してみます


: `V$ENCRYPTION_WALLET <https://docs.oracle.com/en/database/oracle/oracle-database/23/refrn/V-ENCRYPTION_WALLET.html>`__ : ウォレットの状態とTDEウォレットの場所に関する情報を表示  
: `V$ENCRYPTION_KEYS <https://docs.oracle.com/en/database/oracle/oracle-database/23/refrn/V-ENCRYPTION_KEYS.html>`__ : マスターキーの説明属性を表示




