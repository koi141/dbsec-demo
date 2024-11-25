##########################################
Oracle Database 23ai FREEのセットアップ
##########################################

**やること**

+ DB23aiをインストールする&DBを作成する
+ サンプルスキーマ（HR）を作成する

******************************
DB23aiをインストールする
******************************

23aiの提供形式は大きく分けて以下の3つです。

+ コンテナ（docker, podman）
+ VMイメージ（VM Vritual Box）
+ rpmパッケージ（ol8,9, Windows）

各インストール手順は `こちらのドキュメント <https://docs.oracle.com/cd/G11854_01/xeinl/index.html>`__ にありますので、お好みの方式でDB環境を作成ください。

rpmでインストールする場合、 `OCIチュートリアル <https://oracle-japan.github.io/ocitutorials/ai-vector-search/ai-vector102-23aifree-install>`__ でも手順の案内があります。
ご参考にしてみてください。OCIチュートリアルを参考にした場合、表領域の作成手順は不要ですので、その前の手順までを行ってください。



**********************************
サンプルスキーマをダウンロードする
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

