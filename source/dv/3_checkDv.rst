############################################
3. Database Vaultを確認する
############################################

**実施内容**

+ 許可リストを作成する



********************************
SYSユーザー
********************************

ユーザーが作成できないことを確認します。

SQL> create user test;
create user test
*
ERROR at line 1:
ORA-01031: insufficient privileges
Help: https://docs.oracle.com/error-help/db/ora-01031/


note
    ユーザーはユーザー管理者である、C##ACCTMGRユーザーが作成することができます。

レルム内のオブジェクトにアクセスできないことを確認します。

    SQL> select * from hr.regions;
    select * from hr.regions
                    *
    ERROR at line 1:
    ORA-01031: insufficient privileges
    Help: https://docs.oracle.com/error-help/db/ora-01031/


********************************
HRユーザーおよびSALES_APPユーザー
********************************
SYSユーザーではアクセスできなかったREGIONS表にアクセスできることを確認します。

SQL> select * from hr.regions;

 REGION_ID REGION_NAME
---------- -------------------------
        10 Europe
        20 Americas
        30 Asia
        40 Oceania
        50 Africa


********************************
APPユーザー
********************************

許可されたIPアドレスのみでアクセスできることを確認します。

許可されたIPアドレスからのアクセスの場合
==============================================
    SQL> set markup csv on
    SQL> select SYS_CONTEXT('USERENV','IP_ADDRESS');
    "SYS_CONTEXT('USERENV','IP_ADDRESS')"
    "192.168.0.10"

    SQL> select * from hr.regions;
    "REGION_ID","REGION_NAME"
    10,"Europe"
    20,"Americas"
    30,"Asia"
    40,"Oceania"
    50,"Africa"


許可されていないIPアドレスからのアクセスの場合
==============================================

    SQL> set markup csv on
    SQL> select SYS_CONTEXT('USERENV','IP_ADDRESS');
    "SYS_CONTEXT('USERENV','IP_ADDRESS')"
    "192.168.130.169"

    SQL> select * from hr.regions;
    select * from hr.regions
                    *
    ERROR at line 1:
    ORA-47306: 20000: DV_Error: Can only be accessed from a specific IP address
    Help: https://docs.oracle.com/error-help/db/ora-47306/