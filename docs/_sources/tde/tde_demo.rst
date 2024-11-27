###########################
TDE 暗号化デモ
###########################

こちらの手順では表領域に対して暗号化を行い、データファイルを暗号化します。

****************************
データファイルの確認
****************************

作成したHRスキーマを使用します。手順ではUSERS表領域に作成していますが、念のために確認します。

.. code-block:: sql
   :caption: PDBで実行 (sysユーザー)

    -- PDBにSYSユーザーで接続
    SQL> set markup csv on

    -- HRスキーマの表領域を確認
    SQL> select username, default_tablespace from dba_users where username ='HR';
    "USERNAME","DEFAULT_TABLESPACE"
    "HR"      ,"USERS"

    -- USER表領域が格納されるデータファイルのパスを確認
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


****************************
表領域の暗号化を行う
****************************

HR表領域の暗号化を行います。

.. code-block:: sql
    :caption: PDBで実行 (sysユーザー)

    -- 実行時間を計測
    SQL> set timing on

    -- HRスキーマを暗号化
    SQL> alter tablespace users encryption online using 'AES256' encrypt;
    
    Tablespace altered.

    Elapsed: 00:00:07.02

    -- データファイルの暗号化を確認する
    SQL> !strings /opt/oracle/oradata/FREE/FREEPDB1/users01.dbf
    ...
    mK=.
    qM$;
    /[Ur
    N5Y9N
    ZSaKE
    y!Ac
    oH/P
    <a)3:oii
    /`_S
    }l d%
    ...
    
    -- SQLは問題なく実行されることを確認
    SQL> select * from hr.jobs;
    "JOB_ID","JOB_TITLE","MIN_SALARY","MAX_SALARY"
    "AD_PRES","President",20080,40000
    "AD_VP","Administration Vice President",15000,30000
    "AD_ASST","Administration Assistant",3000,6000
    "FI_MGR","Finance Manager",8200,16000
    "FI_ACCOUNT","Accountant",4200,9000
    "AC_MGR","Accounting Manager",8200,16000
    "AC_ACCOUNT","Public Accountant",4200,9000
    "SA_MAN","Sales Manager",10000,20080
    "SA_REP","Sales Representative",6000,12008
    "PU_MAN","Purchasing Manager",8000,15000
    "PU_CLERK","Purchasing Clerk",2500,5500
    "ST_MAN","Stock Manager",5500,8500
    "ST_CLERK","Stock Clerk",2008,5000
    "SH_CLERK","Shipping Clerk",2500,5500
    "IT_PROG","Programmer",4000,10000
    "MK_MAN","Marketing Manager",9000,15000
    "MK_REP","Marketing Representative",4000,9000
    "HR_REP","Human Resources Representative",4000,9000
    "PR_REP","Public Relations Representative",4500,10500

    19 rows selected.

    Elapsed: 00:00:00.02



****************************
暗号化された表領域を復号する
****************************

オンライン暗号化を行いましたが、同様にオンライン復号も行うことができます


.. code-block:: sql
    :caption: PDBで実行 (sysユーザー)

    alter tablespace users encryption online decrypt;

    -- 復号されていることを確認する
    SQL> !strings /opt/oracle/oradata/FREE/FREEPDB1/users01.dbf
    }|{z
    VFREE
    USERS
    ...
    Geneve
    Rua Frei Caneca 1360    01307-002       Sao Paulo       Sao Paulo
    Schwanthalerstr. 7031
    80925
    Munich
    Bavaria
    9702 Chester Road
    09629850293     Stretford
    Manchester
    (Magdalen Centre, The Oxford Science Park

