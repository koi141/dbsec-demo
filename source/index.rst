.. dbsec-demo documentation master file, created by
   sphinx-quickstart on Wed Nov 20 16:08:19 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

dbsec-demo
========================

.. hint::

   このデモ手順で示す実行結果は見やすくなるよう、整形または省略している箇所があります。
   そのため実機での結果と多少異なる場合がありますので、ご了承ください。

test

.. toctree::
   :maxdepth: 2
   :caption: 環境準備:

   /setup/db23ai
   /setup/sampleSchema

.. toctree::
   :maxdepth: 2
   :caption: TDE:
   
   /tde/setup
   /tde/tde_demo

.. toctree::
   :maxdepth: 2
   :caption: NNE:
   
   /nne/setup
   /nne/confirm

.. toctree::
   :maxdepth: 2
   :caption: Data Redaction:
   
   /redact/setup
   /redact/caution