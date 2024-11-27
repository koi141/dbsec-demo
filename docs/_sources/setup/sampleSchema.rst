##########################################
サンプルスキーマ（HR）を作成する
##########################################


**********************************
サンプルスキーマのダウンロード
**********************************

本デモで使用するためのHRスキーマを作成します。

サンプルスキーマとしてGitHubで公開されています `こちら <https://github.com/oracle-samples/db-sample-schemas/archive/refs/tags/v23.3.zip>`__ を使用します。

.. code:: bash

    wget https://github.com/oracle-samples/db-sample-schemas/archive/refs/tags/v23.3.zip

上記を実行すると ``v23.3.zip`` がダウンロードされるので、unzipします。

.. code:: bash

    unzip v23.3.zip

``db-sample-schemas-23.3`` が展開されます。
その配下にある ``/human_resources/hr_install.sql`` を後に実行します。
ファイル名が変わっている場合がありますので、ファイル名およびパスはお手元の環境のもので控えておいてください。

なお、サンプルスキーマの詳細は `こちら <https://docs.oracle.com/cd/F82042_01/comsc/schema-diagrams.html>`__ で示されています。

********************************
サンプルスキーマを作成する
********************************

HRスキーマを作成するため、DBに接続します。

.. code:: bash

    # oracleユーザーにスイッチ
    $ sudo su - oracle

    # sqlplusでDBに接続
    $ sqlplus / as sysdba


.. code:: sql

    -- pdbを確認し、freepdbに接続
    SQL> show pdbs
    SQL> alter session set container = FREEPDB1; 
    SQL> show con_name

先ほどダウンロードしました、 ``db-sample-schemas-23.3/human_resources/hr_install.sql`` を実行します。

.. code-block:: sql
    :emphasize-lines: 8, 10

    SQL> @/home/oracle/db-sample-schemas-23.3/human_resources/hr_install.sql

    Thank you for installing the Oracle Human Resources Sample Schema.
    This installation script will automatically exit your database session
    at the end of the installation or if any error is encountered.
    The entire installation will be logged into the 'hr_install.log' log file.

    Enter a password for the user HR: <HRユーザーのパスワードを入力>

    Enter a tablespace for HR [USERS]:  <HRユーザーのパスワードを再入力>
    Do you want to overwrite the schema, if it already exists? [YES|no]: n
    ******  Creating REGIONS table ....
    ...

正しく作成されていることを確認します。

.. image:: ../_static/db23ai/HR_OEスキーマ.gif

.. code:: sql

    SQL> select table_name from all_tables where owner = 'HR';

    TABLE_NAME
    --------------------------------------------------------------------------------
    COUNTRIES
    REGIONS
    LOCATIONS
    DEPARTMENTS
    JOBS
    EMPLOYEES
    JOB_HISTORY

    7 rows selected.

********************************
日本語対応させる
********************************

.. code:: sql

    SQL> select * from gv$nls_parameters;
    "INST_ID","PARAMETER"        ,"VALUE","CON_ID"
    1        ,"NLS_LANGUAGE"     ,"AMERICAN",1
    1        ,"NLS_TERRITORY","AMERICA",1
    1        ,"NLS_CURRENCY","$",1
    1        ,"NLS_ISO_CURRENCY","AMERICA",1
    1        ,"NLS_NUMERIC_CHARACTERS",".,",1
    1        ,"NLS_CALENDAR","GREGORIAN",1
    1        ,"NLS_DATE_FORMAT","DD-MON-RR",1
    1        ,"NLS_DATE_LANGUAGE","AMERICAN",1
    1        ,"NLS_CHARACTERSET","AL32UTF8",1
    1        ,"NLS_SORT","BINARY",1
    1        ,"NLS_TIME_FORMAT","HH.MI.SSXFF AM",1
    1        ,"NLS_TIMESTAMP_FORMAT","DD-MON-RR HH.MI.SSXFF AM",1
    1,"NLS_TIME_TZ_FORMAT","HH.MI.SSXFF AM TZR",1
    1,"NLS_TIMESTAMP_TZ_FORMAT","DD-MON-RR HH.MI.SSXFF AM TZR",1
    1,"NLS_DUAL_CURRENCY","$",1
    1,"NLS_NCHAR_CHARACTERSET","AL16UTF16",1
    1,"NLS_COMP","BINARY",1
    1,"NLS_LENGTH_SEMANTICS","BYTE",1
    1,"NLS_NCHAR_CONV_EXCP","FALSE",1

    19 rows selected.


