###############################
通信の暗号化を確認する
###############################


ウィンドウを２つ立ち上げます

+ [DBサーバー側] 通信をキャプチャする
+ [クライアント側] DBにログインし、SQLで問い合わせる


************************************
DBサーバー側で通信を待ち受ける
************************************





.. code-block:: bash
    :caption: DBサーバー側で操作

    # ポート1521を解放します
    sudo firewall-cmd --permanent --add-port=1521/tcp
    sudo firewall-cmd --reload

    # インターフェース名と1521番ポートが解放されていることを確認します
    $ sudo firewall-cmd --list-all
    public (active)
        target: default
        icmp-block-inversion: no
        interfaces: enp0s5
        sources:
        services: dhcpv6-client ssh
        ports: 1521/tcp
        protocols:
        forward: no
        masquerade: no
        forward-ports:
        source-ports:
        icmp-blocks:
        rich rules:


    # インターフェースの番号を確認
    $ sudo tshark -D
    Running as user "root" and group "root". This could be dangerous.
    1. enp0s5
    2. lo (Loopback)
    3. any
    4. bluetooth-monitor
    5. nflog
    6. nfqueue
    7. usbmon0
    8. usbmon1
    9. usbmon2
    10. usbmon3
    ...


TCP/1521で受け取る通信をキャプチャします。
同時にDBに接続している環境では、 ``'ip.addr==X.X.X.X and tcp.port==1521'`` として、クライアントのIPアドレスでさらに制限をかけるといいでしょう。

.. code:: bash
    
    $ sudo tshark -i 1 -Y 'tcp.port==1521' -x



端末をそのままにしておきながら、次の手順でDBに接続します。


************************************
クライアント側でDBに接続する
************************************


簡易接続子を使います

.. code:: bash

    sqlplus hr/<パスポート>@<接続先ホスト名>:<ポート番号>/<サービス名>

    sqlplus hr/Welcome1#Welcome1#@192.168.130.169:1521/freepdb1

    $ sqlplus hr@192.168.130.169:1521/freepdb1

    SQL*Plus: Release 21.0.0.0.0 - Production on Tue Nov 26 22:09:52 2024
    Version 21.11.0.0.0

    Copyright (c) 1982, 2022, Oracle.  All rights reserved.

    Enter password: <パスワードを入力>
    Last Successful login time: Tue Nov 26 2024 22:08:58 +09:00

    Connected to:
    Oracle Database 23ai Free Release 23.0.0.0.0 - Develop, Learn, and Run for Free
    Version 23.6.0.24.10

    SQL> show user con_name
    USER is "HR"

    CON_NAME
    ------------------------------
    FREEPDB1

    -- jobs表を取得
    SQL> select * from jobs;

    JOB_ID     JOB_TITLE                           MIN_SALARY MAX_SALARY
    ---------- ----------------------------------- ---------- ----------
    AD_PRES    President                                20080      40000
    AD_VP      Administration Vice President            15000      30000
    AD_ASST    Administration Assistant                  3000     6000
    FI_MGR     Finance Manager                           8200      16000
    FI_ACCOUNT Accountant                                4200     9000
    AC_MGR     Accounting Manager                        8200      16000
    ...


この状態でDBサーバーでパケットキャプチャしていた端末を見ると、以下のようにjobsテーブルの内容が平文でやり取りされていることを確認できます。

.. code:: text

    （一部抜粋）
    ...
    0080  2c 01 04 05 41 44 5f 56 50 1d 41 64 6d 69 6e 69   ,...AD_VP.Admini
    0090  73 74 72 61 74 69 6f 6e 20 56 69 63 65 20 50 72   stration Vice Pr
    00a0  65 73 69 64 65 6e 74 03 c3 02 33 02 c3 04 07 2a   esident...3....*
    00b0  2c 01 04 07 41 44 5f 41 53 53 54 18 41 64 6d 69   ,...AD_ASST.Admi
    00c0  6e 69 73 74 72 61 74 69 6f 6e 20 41 73 73 69 73   nistration Assis
    00d0  74 61 6e 74 02 c2 1f 02 c2 3d 07 21 2c 01 04 06   tant.....=.!,...
    00e0  46 49 5f 4d 47 52 0f 46 69 6e 61 6e 63 65 20 4d   FI_MGR.Finance M
    00f0  61 6e 61 67 65 72 02 c2 53 03 c3 02 3d 07 1f 2c   anager..S...=..,
    0100  01 04 0a 46 49 5f 41 43 43 4f 55 4e 54 0a 41 63   ...FI_ACCOUNT.Ac
    0110  63 6f 75 6e 74 61 6e 74 02 c2 2b 02 c2 5b 07 24   countant..+..[.$
    0120  2c 01 04 06 41 43 5f 4d 47 52 12 41 63 63 6f 75   ,...AC_MGR.Accou
    0130  6e 74 69 6e 67 20 4d 61 6e 61 67 65 72 02 c2 53   nting Manager..S
    0140  03 c3 02 3d 07 26 2c 01 04 0a 41 43 5f 41 43 43   ...=.&,...AC_ACC
    0150  4f 55 4e 54 11 50 75 62 6c 69 63 20 41 63 63 6f   OUNT.Public Acco
    0160  75 6e 74 61 6e 74 02 c2 2b 02 c2 5b 07 20 2c 01   untant..+..[. ,.
    0170  04 06 53 41 5f 4d 41 4e 0d 53 61 6c 65 73 20 4d   ..SA_MAN.Sales M
    0180  61 6e 61 67 65 72 02 c3 02 04 c3 03 01 51 07 27   anager.......Q.'
    0190  2c 01 04 06 53 41 5f 52 45 50 14 53 61 6c 65 73   ,...SA_REP.Sales
    01a0  20 52 65 70 72 65 73 65 6e 74 61 74 69 76 65 02    Representative.
    01b0  c2 3d 04 c3 02 15 09 07 24 2c 01 04 06 50 55 5f   .=......$,...PU_
    01c0  4d 41 4e 12 50 75 72 63 68 61 73 69 6e 67 20 4d   MAN.Purchasing M
    01d0  61 6e 61 67 65 72 02 c2 51 03 c3 02 33 07 23 2c   anager..Q...3.#,
    01e0  01 04 08 50 55 5f 43 4c 45 52 4b 10 50 75 72 63   ...PU_CLERK.Purc
    01f0  68 61 73 69 6e 67 20 43 6c 65 72 6b 02 c2 1a 02   hasing Clerk....
    0200  c2 38 07 1e 2c 01 04 06 53 54 5f 4d 41 4e 0d 53   .8..,...ST_MAN.S
    ...


************************************
通信の暗号化設定を行う
************************************

DBサーバーの ``$ORACLE_HOME/network/admin`` にある ``sqlnet.ora`` ファイルを編集します。

.. code:: bash

    $ ls $ORACLE_HOME/network/admin
    listener.ora  samples  shrept.lst  sqlnet.ora  tnsnames.ora

    $ vi $ORACLE_HOME/network/admin/sqlnet.ora
    # sqlnet.ora Network Configuration File: /opt/oracle/product/23ai/dbhomeFree/network/admin/sqlnet.ora
    # Generated by Oracle configuration tools.

    NAMES.DIRECTORY_PATH= (TNSNAMES, EZCONNECT)

