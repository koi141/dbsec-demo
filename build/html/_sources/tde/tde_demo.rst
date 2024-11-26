###########################
TDE 暗号化デモ
###########################

こちらの手順では表領域に対して暗号化を行い、データファイルを暗号化します。

****************************
データファイルの確認
****************************

作成したHRスキーマを使用します。手順ではUSERS表領域に作成していますが、念のために確認します。

.. code:: sql

    -- PDBにSYSユーザーで接続
    SQL> set markup csv on
    SQL> select username, default_tablespace from dba_users where username ='HR';
    "USERNAME","DEFAULT_TABLESPACE"
    "HR"      ,"USERS"

    -- データファイルのパスを確認
    SQL> select tablespace_name, file_name from dba_data_files where tablespace_name = 'USERS';
    "TABLESPACE_NAME","FILE_NAME"
    "USERS"          ,"/opt/oracle/oradata/FREE/FREEPDB1/users01.dbf"

    -- データファイルの中身を確認
    SQL> !strings /opt/oracle/oradata/FREE/FREEPDB1/users01.dbf
    }|{z
    VFREE
    USERS
    ...
    CJOHNSON
    44.1632.960034
    SA_REP
    O       Kimberely
    Grant
    KGRANT
    44.1632.960033
    SA_REP
    Jack
    Livingston
    JLIVINGS
    44.1632.960032
    ...


このようにTDEで暗号化されていないと、stringsコマンドでデータの中身が見えてしまうことが分かります。



.. code:: sql

    ALTER TABLESPACE crm ENCRYPTION ONLINE USING 'AES256' ENCRYPT;
    