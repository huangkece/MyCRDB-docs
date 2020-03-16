教程
=========================

1.搭建环境
-------------------------
下载CockroachDB源码并解压::

    wget -qO- https://binaries.cockroachdb.com/cockroach-v2.0.5.src.tgz | tar  xvz

切换到源码根目录，用make工具进行编译::

    cd cockroach-v2.0.5
    make build


2.启动集群
-------------------------
非安全模式::

    cockroach start --insecure --host=localhost

安全模式::

    cockroach start --certs-dir=certs --host=localhost --http-host=localhost

3.架构图
-------------------------

.. image:: ../img/db_structure.jpeg

