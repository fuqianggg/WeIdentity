
.. _weidentity-rest-deploy:

WeIdentity RestService 部署文档
================================

Server部署说明
---------------

RestService会提供一个开源的Server组件包以供部署。

环境要求
^^^^^^^^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 30 30 60

   * - 物料
     - 版本
     - 说明
   * - CentOS/Ubuntu
     - 7.2 / 16.04，64位
     - 部署RestServer用
   * - JDK
     - 1.8+
     - 推荐使用1.8u141及以上
   * - FISCO-BCOS
     - 1.3.6+
     - 确保它可以和RestServer机器互相telnet联通其channelPort端口
   * - gradle
     - 4.6+
     - 
   * - WeIdentity Java SDK
     - 1.1.1
     - 

物料准备
^^^^^^^^^^^^^

将dist.zip解压可以得到：

- dist/app/ 启动jar包
- dist/lib/ 依赖库
- dist/conf/ 配置文件
- 当前目录下的其他命令脚本

修改配置文件
^^^^^^^^^^^^^

- 首先，确认WeIdentity Java SDK已部署完毕，并确认其配置文件中的四个合约地址可访问。
- 修改dist/conf/ApplicationContext.xml，更新合约地址及区块链节点信息。
- 修改dist/conf/application.properties，确认HTTP端口地址及重定向地址已设置且未被其他程序占用。

启动/停止应用
^^^^^^^^^^^^^

进入dist/目录执行：

- start.sh 启动应用
- stop.sh 停止应用

轻客户端使用说明
-----------------

RestService会同时提供一个完全开源的轻客户端以供示例调用。

环境要求
^^^^^^^^^

轻客户端仅需要一个java运行环境。

物料准备
^^^^^^^^^^^^^

将dist.zip解压可以得到：

- dist/app/ 启动jar包
- dist/conf/ 配置文件

修改配置文件
^^^^^^^^^^^^^

用户自己的私钥、RestService的端口和地址需要配置在dist/conf/application.properties内

接下来您便可以调用HTTP POST/GET及各类Sign方法了。