.. dbsec-demo documentation master file, created by
   sphinx-quickstart on Wed Nov 20 16:08:19 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

############################################
dbsec-demo
############################################

.. danger:: **ページリンク移転について**

   サイトは以下のリンクに変更しました。（ `新規サイト <https://koi141.github.io/demo-dbsec/>`__ はこちら）
   
   .. code::

      https://koi141.github.io/demo-dbsec/

   | 2025年3月末にはこちらのページはアクセスできなくなる予定ですのでご注意ください
   | 特に内容に変更はありませんが、今後は新規サイトにコンテンツ追加を行っていきます。


.. hint::

   このデモ手順では、実行結果の一部を見やすくするために整形や省略を行っています。
   そのため、実際の結果とは若干異なる場合がありますので、ご了承ください。

.. toctree::
   :maxdepth: 2
   :caption: 環境準備:

   /env_setup/1_db23ai
   /env_setup/2_sampleSchema

.. toctree::
   :maxdepth: 2
   :caption: TDE（透過的データ暗号化）:
   
   /tde/1_setup
   /tde/2_encryption
   /tde/3_autoWalletOpen

.. toctree::
   :maxdepth: 2
   :caption: ネイティブ・ネットワーク暗号化:
   
   /nne/1_setup
   /nne/2_encryption

.. toctree::
   :maxdepth: 2
   :caption: Data Redaction:
   
   /redact/1_setup
   /redact/2_redaction
   /redact/3_note

.. toctree::
   :maxdepth: 2
   :caption: Virtual Private Database:
   
   /vpd/1_rowControl
   /vpd/2_columnControl
   /vpd/3_clientIdentifier
   /vpd/4_cleanup


.. toctree::
   :maxdepth: 2
   :caption: SQL Firewall:
   
   /sqlfirewall/1_setup
   /sqlfirewall/2_learningTraffic
   /sqlfirewall/3_allowTraffic
   /sqlfirewall/4_checkFirewall

.. toctree::
   :maxdepth: 2
   :caption: Database Vault:
   
   /dv/1_setup
   /dv/2_authRealm
   /dv/3_checkDv
   /dv/4_twoPersonIntegrity
   /dv/5_cleanup
   /dv/99_checkComponent

.. toctree::
   :maxdepth: 2
   :caption: Oracle Label Security:
   
   /ols/1_setup
   /ols/2_setupPolicy
   /ols/3_checkOls
   /ols/4_labelColumn
   /ols/5_cleanup