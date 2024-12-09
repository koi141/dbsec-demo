############################################
3. ラベル制御の確認
############################################


**実施内容**

+ HRユーザーで接続する
+ SALES_APPユーザーで接続する



****************************
HRユーザーで接続する
****************************

SQL> select * from HR.JOB_HISTORY_4OLS;

EMPLOYEE_ID START_DAT END_DATE  JOB_ID     DEPARTMENT_ID OLS_LABEL_DEMO
----------- --------- --------- ---------- ------------- --------------
        102 13-JAN-11 24-JUL-16 IT_PROG               60            200
        101 21-SEP-07 27-OCT-11 AC_ACCOUNT           110            300
        101 28-OCT-11 15-MAR-15 AC_MGR               110            400
        201 17-FEB-14 19-DEC-17 MK_REP                20            300
        114 24-MAR-16 31-DEC-17 ST_CLERK              50            200
        122 01-JAN-17 31-DEC-17 ST_CLERK              50            200
        200 17-SEP-05 17-JUN-11 AD_ASST               90            200
        176 24-MAR-16 31-DEC-16 SA_REP                80            300
        176 01-JAN-17 31-DEC-17 SA_MAN                80            400
        200 01-JUL-12 31-DEC-16 AC_ACCOUNT            90            300

10 rows selected.

SQL> col label for a20
SQL> select JOB_ID, LABEL_TO_CHAR(OLS_LABEL_DEMO) LABEL  from HR.JOB_HISTORY_4OLS;

JOB_ID     LABEL
---------- --------------------
IT_PROG    INTL
AC_ACCOUNT CONF
AC_MGR     SENS
MK_REP     CONF
ST_CLERK   INTL
ST_CLERK   INTL
AD_ASST    INTL
SA_REP     CONF
SA_MAN     SENS
AC_ACCOUNT CONF

10 rows selected.


****************************
SALES_APPユーザーで接続する
****************************


SQL> select * from HR.JOB_HISTORY_4OLS;

EMPLOYEE_ID START_DAT END_DATE  JOB_ID     DEPARTMENT_ID OLS_LABEL_DEMO
----------- --------- --------- ---------- ------------- --------------
        102 13-JAN-11 24-JUL-16 IT_PROG               60            200
        101 21-SEP-07 27-OCT-11 AC_ACCOUNT           110            300
        201 17-FEB-14 19-DEC-17 MK_REP                20            300
        114 24-MAR-16 31-DEC-17 ST_CLERK              50            200
        122 01-JAN-17 31-DEC-17 ST_CLERK              50            200
        200 17-SEP-05 17-JUN-11 AD_ASST               90            200
        176 24-MAR-16 31-DEC-16 SA_REP                80            300
        200 01-JUL-12 31-DEC-16 AC_ACCOUNT            90            300

8 rows selected.

SQL> col label for a20
SQL> select JOB_ID, LABEL_TO_CHAR(OLS_LABEL_DEMO) LABEL  from HR.JOB_HISTORY_4OLS;

JOB_ID     LABEL
---------- --------------------
IT_PROG    INTL
AC_ACCOUNT CONF
MK_REP     CONF
ST_CLERK   INTL
ST_CLERK   INTL
AD_ASST    INTL
SA_REP     CONF
AC_ACCOUNT CONF

8 rows selected.