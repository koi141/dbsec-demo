.. dbsec-demo documentation master file, created by
   sphinx-quickstart on Wed Nov 20 16:08:19 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

dbsec-demo
========================

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