---
layout: post
category: swoole
tags: [swoole , php]
---

#Swoole 之旅

######出发之前：因为一个论坛项目的mysql压力太大，决定引入swoole的连接池来看看效果，因此开始这段Swoole之旅

##Swoole install

要用swoole必须要先有PHP的环境，本机的环境是php5.5,swoole的版本是1.7.8 stable版本

swoole扩展的[下载地址](https://github.com/swoole/swoole-src/releases),alpha版本用于实验新的特性，stable为稳定版本

将下载的源码进行解压安装，执行如下命令

    cd swoole-src-swoole-1.7.6-stable/
    phpize
    ./configure --enable-async-mysql
    sudo make
    sudo make install

####出现问题：

在make安装的时候出现错误，提示swoole加--enable-async-mysql后的是依赖于mysqli或者mysqlnd的

但是在php -m| grep mysqli 的时候可以看到mysqli
    
####解决方法

后发现是我安装php的mysql扩展的时候没有使用mysqlnd的方式进行安装

这就需要重新安装php程序

在安装PHP的过程中加入

    --with-mysql=mysqlnd 
    --with-mysqli=mysqlnd
    --with-pdo-mysql=mysqlnd 

说明以mysqlnd的方式安装mysql扩展

然后在php.ini中加入

    extension=swoole.so

可以在终端通过命令

    php -m

查看扩展的情况，如果有swoole就说明安装成功了

下面是一个php ECHO服务器的实例

    // Server
    class Server
    {
        private $serv;

        public function __construct() {
            $this->serv = new swoole_server("0.0.0.0", 9501);
            $this->serv->set(array(
                        'worker_num' => 8,
                        'daemonize' => false,
                        'max_request' => 10000,
                        'dispatch_mode' => 2,
                        'debug_mode'=> 1
                        ));

            $this->serv->on('Start', array($this, 'onStart'));
            $this->serv->on('Connect', array($this, 'onConnect'));
            $this->serv->on('Receive', array($this, 'onReceive'));
            $this->serv->on('Close', array($this, 'onClose'));

            $this->serv->start();
        }

        public function onStart( $serv ) {
            echo "Start\n";
        }

        public function onConnect( $serv, $fd, $from_id ) {
            $serv->send( $fd, "Hello {$fd}!" );
        }

        public function onReceive( swoole_server $serv, $fd, $from_id, $data ) {
            echo "Get Message From Client {$fd}:{$data}\n";
        }

        public function onClose( $serv, $fd, $from_id ) {
            echo "Client {$fd} close connection\n";
        }
    }
    // 启动服务器
    $server = new Server();

下面是客户端

    <?php
    class Client
    {
        private $client;

        public function __construct() {
            $this->client = new swoole_client(SWOOLE_SOCK_TCP);
        }

        public function connect() {
            if( !$this->client->connect("127.0.0.1", 9501 , 1) ) {
                echo "Error: {$fp->errMsg}[{$fp->errCode}]\n";
            }
            $message = $this->client->recv();
            echo "Get Message From Server:{$message}\n";

            fwrite(STDOUT, "请输入消息：");  
            $msg = trim(fgets(STDIN));
            $this->client->send( $msg );
        }
    }

    $client = new Client();
    $client->connect();

使用方法：

1.先运行server文件： php server.php

2.再建一个窗口，执行：php client.php

就可以进行会话了。

##Swoole的基本理解

swoole是基于事件的出发的，目前接触的事件有

    onstart:回调函数在server被调用之前运行

    onConnect：在新客户端连接时进行调用

    onReceive:在有数据发送到server时进行调用

    onClose：在有客户端断开时被调用

在客户端可以通过recv()和send()的两个函数来接受和发送请求

##swoole的异步任务Task

###开启task功能

在配置项中添加task_worker_num就可以了

    $serv->set(array(
        'task_worker_num' => 8
        ));

就开启了task功能，再给swoole绑定2个回调函数onTask和onFinish,2个任务分别用于执行Task任务和处理Task任务返回的结果

###使用Task

首先发起一个Task，

    public function onReceive( swoole_server $serv, $fd, $from_id, $data ){
        echo "Get Message From Client {$fd}:{$data}\n";
        // send a task to task worker.
        $param = array(
                'fd' => $fd
                );
        // start a task
        $serv->task( json_encode( $param ) );

        echo "Continue Handle Worker\n";
    }

直接调用task方法就可以发起一个任务，swoole会将请求交给task_work,当前的worker进程会继续进行

当一个任务发起后task_work进程会响应onTask回调函数，如下

    public function onTask($serv,$task_id,$from_id, $data) {
        echo "This Task {$task_id} from Worker {$from_id}\n";
        echo "Data: {$data}\n";
        for($i = 0 ; $i < 10 ; $i ++ ) {
            sleep(1);
            echo "Task {$task_id} Handle {$i} times...\n";
        }
        $fd = json_decode( $data , true )['fd'];
        $serv->send( $fd , "Data in Task {$task_id}");
        return "Task {$task_id}'s result";
    }

在onTask回调中，可以通过task_id和from_id来区分不同的进程的不同的task,当一个task结束后，通过return一个字符串将执行结果返回给worker进程，worker进程将通过OnFinish回调函数接受这个处理结果

Finish回调

    public function onFinish($serv,$task_id, $data) {
        echo "Task {$task_id} finish\n";
        echo "Result: {$data}\n";
    }
在OnFinish回调中，会接受到Task任务的处理结果$data.

###swoole_client

创建一个异步的client的代码

    $client = new swoole_client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);

    SWOOLE_SOCK_ASYNC表明是一个异步的client

异步的client的回调函数

    $client->on("connect", function($cli) {
            $cli->send("hello world\n");
    });
    $client->on("receive", function($cli, $data){
            echo "Received: ".$data."\n";
    });
    $client->on("error", function($cli){
            echo "Connect failed\n";
    });
    $client->on("close", function($cli){
            echo "Connection close\n";
    });

##Time定时器，心跳检测及mysql连接池

Swoole提供了一个内置的Timer定时器功能，可以通过addtimer可以在Swoole中添加一个定时器，该定时器会在预定的时间时，调用回调函数onTime

    $this->serv->on('Timer', array($this, 'onTimer'));

        public function onWorkerStart( $serv , $worker_id) {
            // 在Worker进程开启时绑定定时器
            // 只有当worker_id为0时才添加定时器,避免重复添加
            if( $worker_id == 0 ) {
                $serv->addtimer(500);
                $serv->addtimer(1000);
                $serv->addtimer(1500);
            }
        }

    public function onTimer($serv, $interval) {
        switch( $interval ) {
            case 500: { // 
                      echo "Do Thing A at interval 500\n";
                      break;
                      }
             case 1000:{
                      echo "Do Thing B at interval 1000\n";
                      break;
                      }
             case 1500:{
                      echo "Do Thing C at interval 1500\n";
                      break;
                      }
        }
    }

##心跳检测

开启心跳检测设置方式

    $this->serv->set(
            array(
                'heartbeat_check_interval' => 60,
                'heartbeat_idle_time' => 600,
                )
            );

其中heartbeat_idle_time的默认值是heartbeat_check_interval的两倍。

在设置这两个选项后，swoole会在内部启动一个线程，每隔heartbeat_check_interval秒后遍历一次全部连接，检查最近一次发送数据的时间和当前时间的差，如果这个差值大于heartbeat_idle_time，则会强制关闭这个连接，并通过回调onClose通知Server进程。

tip1：结合之前的Timer功能，如果我们想维持连接，就设置一个略小于如果这个差值大于heartbeat_idle_time的定时器，在定时器内向所有连接发送一个心跳包。如果收到心跳回应，则判断连接正常，如果没有收到，则关闭这个连接或者再次尝试发送。

tip2: 一定要bind后在用addtimer进行加载；

##mysql连接池

在PHP中，访问MySQL数据库往往是性能提升的瓶颈。

而MySQL连接池我想大家都不陌生，这是一个很好的提升数据库访问性能的方式。

传统的MySQL连接池，是预先申请一定数量的连接，每一个新的请求都会占用其中一个连接，请求结束后再将连接放回池中，如果所有连接都被占用，新来的连接则会进入等待状态。

知道了MySQL连接池的实现原理，那我们来看如何使用Swoole实现一个连接池。

首先，Swoole允许开启一定量的Task Worker进程，我们可以让每个进程都拥有一个MySQL连接，并保持这个连接，这样，我们就创建了一个连接池。

其次，设置swoole的dispatch_mode为抢占模式(主进程会根据Worker的忙闲状态选择投递，只会投递给处于闲置状态的Worker)。

这样，每个task都会被投递给闲置的Task Worker。

这样，我们保证了每个新的task都会被闲置的Task Worker处理，如果全部Task Worker都被占用，则会进入等待队列。























