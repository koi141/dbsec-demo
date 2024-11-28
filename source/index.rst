.. dbsec-demo documentation master file, created by
   sphinx-quickstart on Wed Nov 20 16:08:19 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

dbsec-demo
========================

.. hint::

   このデモ手順では、実行結果の一部を見やすくするために整形や省略を行っています。
   そのため、実際の結果とは若干異なる場合がありますので、ご了承ください。

test

.. toctree::
   :maxdepth: 2
   :caption: 環境準備:

   /demo_env_setup/db23ai
   /demo_env_setup/sampleSchema

.. toctree::
   :maxdepth: 2
   :caption: TDE（透過的データ暗号化）:
   
   /tde/setup
   /tde/tde_demo

.. toctree::
   :maxdepth: 2
   :caption: ネイティブ・ネットワーク暗号化:
   
   /nne/setup
   /nne/confirm

.. toctree::
   :maxdepth: 2
   :caption: Data Redaction:
   
   /redact/setup
   /redact/caution