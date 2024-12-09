############################################
4. ラベルデータの処理
############################################



****************************
データ・ラベルの表示
****************************

見ていただくとわかる通りですが、ラベル列の実態は見た通り、数字になりますので、不等号で結果を絞ることもできます。

    SQL> select JOB_ID, OLS_LABEL_DEMO from HR.JOB_HISTORY_4OLS where OLS_LABEL_DEMO >= 300;

    JOB_ID     OLS_LABEL_DEMO
    ---------- --------------
    AC_ACCOUNT            300
    AC_MGR                400
    MK_REP                300
    SA_REP                300
    SA_MAN                400
    AC_ACCOUNT            300

    6 rows selected.

ORDER BYでソートすることもできます。

    SQL> select JOB_ID, OLS_LABEL_DEMO from HR.JOB_HISTORY_4OLS order by OLS_LABEL_DEMO;

    JOB_ID     OLS_LABEL_DEMO
    ---------- --------------
    IT_PROG               200
    ST_CLERK              200
    AD_ASST               200
    ST_CLERK              200
    AC_ACCOUNT            300
    SA_REP                300
    MK_REP                300
    AC_ACCOUNT            300
    SA_MAN                400
    AC_MGR                400

    10 rows selected.


****************************
ラベル列の非表示
****************************

HIDEオプションを表に適用することで、ポリシーを表す列を非表示にするように選択できます。

HIDEポリシー構成オプションを指定できるのは、表にOracle Label Securityのポリシーを初めて適用するとき(つまり、表にポリシー列を追加するとき)です。このようにすると、ポリシーのラベルを含む列は表示されません。
ポリシーを適用した後は、DROP_COLUMNパラメータをTRUEに設定してそのポリシーを削除しないかぎり、列の非表示(または表示)ステータスは変更できません。
その後は、ポリシーを新規の非表示ステータスで再適用できます。

一旦表に紐づけられている、既存ポリシーを削除します。


BEGIN
    SA_POLICY_ADMIN.REMOVE_TABLE_POLICY (
        policy_name     => 'OLS_POL_DEMO',
        schema_name     => 'HR',
        table_name      => 'JOB_HISTORY_4OLS',
        drop_column     => TRUE
    );
END;
/

ラベル列が削除されたことを確認します

    SQL> desc HR.JOB_HISTORY_4OLS;
    Name                                      Null?    Type
    ----------------------------------------- -------- ----------------------------
    EMPLOYEE_ID                               NOT NULL NUMBER(6)
    START_DATE                                NOT NULL DATE
    END_DATE                                  NOT NULL DATE
    JOB_ID                                    NOT NULL VARCHAR2(10)
    DEPARTMENT_ID                                      NUMBER(4)



BEGIN
    SA_POLICY_ADMIN.APPLY_TABLE_POLICY (
        policy_name    => 'OLS_POL_DEMO',
        schema_name    => 'HR', 
        table_name     => 'JOB_HISTORY_4OLS',
        table_options  => 'READ_CONTROL, HIDE');
END;
/

SQL> desc HR.JOB_HISTORY_4OLS;
 Name                                      Null?    Type
 ----------------------------------------- -------- ----------------------------
 EMPLOYEE_ID                               NOT NULL NUMBER(6)
 START_DATE                                NOT NULL DATE
 END_DATE                                  NOT NULL DATE
 JOB_ID                                    NOT NULL VARCHAR2(10)
 DEPARTMENT_ID                                      NUMBER(4)

SQL> select * from HR.JOB_HISTORY_4OLS;

EMPLOYEE_ID START_DAT END_DATE  JOB_ID     DEPARTMENT_ID
----------- --------- --------- ---------- -------------
        102 13-JAN-11 24-JUL-16 IT_PROG               60
        101 21-SEP-07 27-OCT-11 AC_ACCOUNT           110
...

しかし、明示的に列名を指定するとラベル列が存在していることが分かります。

    SQL> select JOB_ID, OLS_LABEL_DEMO from HR.JOB_HISTORY_4OLS;

    JOB_ID     OLS_LABEL_DEMO
    ---------- --------------
    IT_PROG
    AC_ACCOUNT
    AC_MGR
    MK_REP
    ...


ラベル列に何もないのはラベル列を一旦削除したためです。
手順２のラベルデータを挿入し直すと以下のように確認できます。

    SQL> select JOB_ID, OLS_LABEL_DEMO from HR.JOB_HISTORY_4OLS;

    JOB_ID     OLS_LABEL_DEMO
    ---------- --------------
    IT_PROG               200
    AC_ACCOUNT            300
    AC_MGR                400
    MK_REP                300