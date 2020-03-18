教程
=========================

1.搭建环境
-------------------------
下载CockroachDB源码并解压::

    wget -qO- https://binaries.cockroachdb.com/cockroach-v2.0.5.src.tgz | tar  xvz

切换到源码根目录，用make工具进行编译::

    cd cockroach-v2.0.5
    make build

2.制作证书
-------------------------
1.创建本地证书目录::

    mkdir certs;
    mkdir my_safe_directory


2.创建ca证书::

    cockroach cert create-ca \--certs-dir=certs \--ca-key=my-safe-directory/ca.key


3.创建节点证书::

    cockroach cert create-node \
    <node1 internal IP address> \
    <node1 external IP address> \
    <node1 hostname> \
    <other common names for node1> \
    localhost \
    127.0.0.1 \
    <load balancer IP address> \
    <load balancer hostname> \
    <other common names for load balancer instances> \--certs-dir=certs \--ca-key=my-safe-directory/ca.key


4.创建节点证书目录::

    ssh <username>@<node1 address> "mkdir certs"


5.复制本地证书到节点证书目录::

    scp certs/ca.crt \
    certs/node.crt \
    certs/node.key \
    <username>@<node1 address>:~/certs


6.删除本地证书::

    rm certs/node.crt certs/node.key


7.重复2-6步骤直到把所有节点的证书都生成并考到响应位置，最后一个节点可以省略第6步。
8.创建客户端证书给root用户::

    cockroach cert create-client \
    root \--certs-dir=certs \--ca-key=my-safe-directory/ca.key


9.到各个节点启动进程::

    cockroach start \--certs-dir=certs --store=node1 \--advertise-addr=<node1 address> \--join=<node1 address>:26257,<node2 address>:26257,<node3 address>:26257 \
    --cache=.25 \ --max-sql-memory=.25 \ --background


3.启动集群
-------------------------
非安全模式::

    cockroach start --insecure --host=localhost

安全模式::

    cockroach start --certs-dir=certs --host=localhost --http-host=localhost

4.移除和退役节点
-------------------------
1.移除节点::

    #非安全模式
    cockroach quit --decommission --insecure --host=<address of node to remove>

    #安全模式
    cockroach quit --decommission --certs-dir=certs --host=<address of node to remove>

2.退役节点::

    #非安全模式
    cockroach node decommission <节点序号> --insecure --host=<address of live node>

    #安全模式
    cockroach node decommission <节点序号> --certs-dir=certs --host=<address of live node>

5.测试tpcc
-------------------------
1.初始化tpcc::

    cockroach workload init tpcc --warehouses=100 'postgresql://root@10.1.163.100:26257/tpcc?sslmode=require&sslrootcert=/root/certs/ca.crt&sslcert=/root/certs/client.root.crt&sslkey=/root/certs/client.root.key'

2.运行::

    nohup cockroach workload run tpcc --duration=2h --warehouses=100 'postgresql://root@10.1.163.100:26257/tpcc?sslmode=require&sslrootcert=/root/certs/ca.crt&sslcert=/root/certs/client.root.crt&sslkey=/root/certs/client.root.key' &> test_0113.log &
    