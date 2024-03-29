> 建议先看官方文档

##### 安装指南
- golang
[安装包下载](https://golang.google.cn/dl/)
```
#安装golang
cd /tmp
wget -c https://dl.google.com/go/go1.16.4.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.12.5.linux-amd64.tar.gz

#升级 
sudo rm -rf /usr/local/go
tar -C /usr/local -xzf /var/container/go1.13.5.linux-amd64.tar.gz

#添加环境变理 ubuntu  vi ~/.bashrc
vim ~/.bash_profile

export PATH=$PATH:/usr/local/go/bin:/root/go/bin
export GOPATH=/root/go
export GO111MODULE=on 
export GOPROXY=https://goproxy.io

source ~/.bash_profile
```
- go-mirco
```
go get github.com/asim/go-micro/v3
```
- protoc
```
#linux
sudo yum install autoconf automake libtool curl make g++ unzip
#ubuntu sudo apt-get install autoconf automake libtool curl make g++ unzip
git clone https://github.com/protocolbuffers/protobuf.git /tmp/protobuf
cd /tmp/protobuf
git submodule update --init --recursive
./autogen.sh

./configure
make
make check
sudo make install
sudo ldconfig # refresh shared library cache.

**go get github.com/golang/protobuf/{proto,protoc-gen-go}**
go get github.com/micro/protoc-gen-micro

#window10
https://github.com/protocolbuffers/protobuf/releases/download/v3.9.0/protoc-3.9.0-win64.zip
```
- 工具包
```
go get github.com/asim/go-micro/cmd/protoc-gen-micro/v3
```

- Etcd or Consul
[官方安装指南](https://www.consul.io/docs/install/index.html)
```
yum install -y etcd 

or

mkdir -p $GOPATH/src/github.com/hashicorp
git clone https://github.com/hashicorp/consul.git  $GOPATH/src/github.com/hashicorp/consul
cd $GOPATH/src/github.com/hashicorp/consul
make tools
make dev
consul -v

consul agent -dev -ui -client='0.0.0.0'
http://{{ipaddress}}:8500

go run main.go --registry=consul  #现默认为mdns
#20200107 新版需引入 _ "github.com/micro/go-plugins/registry/consul" 否则无法用consul
```
- 测试
[官方示例](https://micro.mu/docs/go-micro.html#run-service)
```
micro new foo #生成一个微服务项目示例

#生成
.
├── main.go
├── plugin.go
├── handler
│   └── example.go
├── subscriber
│   └── example.go
├── proto/example
│   └── example.proto
├── Dockerfile
├── Makefile
├── README.md
└── go.mod


download protobuf for micro:

brew install protobuf
go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
go get -u github.com/micro/protoc-gen-micro

compile the proto file example.proto:

cd /root/go/src/foo
protoc --proto_path=. --go_out=. --micro_out=. proto/example/example.proto

go run main.go --registry=consul
go run client.go --registry=consul
```

官方示例测试
```
#启动微服务,服务发现使用默认的mdns
go run ./srv/main.go
#go run main.go --registry=consul --registry_address=192.168.1.76:8500

#启动网关
micro api --handler=api --namespace=go.micro.srv  --enable_rpc=true （一定要加这个参数，才能通过/rpc方式调用，手册上没讲明，没带这个参数就直接访问，误导了）
#micro --registry=consul --registry_address=192.168.1.76:8500  api --handler=api --namespace=go.micro.srv  --enable_rpc=true

#访问 
#向micro api发起http请求
HTTP请求的路径/greeter/say/hello会被路由到服务go.micro.api.greeter的方法Say.Hello上,Say与proto中定义的服务名称一致
go run api.go --registry=consul --registry_address=192.168.1.76:8500
http://localhost:8080/greeter/Say/Hello
http://localhost:8080/greeter/say/hello?name=sam

#绕过 micro web、micro api 等 直接访问servcies,(自测找不到服务 启动API时需加参数 --enable_rpc=true)
curl -H 'Content-Type: application/json' \
     -d '{"service": "go.micro.srv.greeter", "method": "Say.Hello", "request": {"name": "John"}}' \
     http://localhost:8080/rpc
```

```
#通过客户端访问微服务
go run ./cli/main.go


三层的服务架构：
- micro api: (localhost:8080) - http访问入口
- api service: (go.micro.api.greeter) - 对外暴露的API服务
- backend service: (go.micro.srv.greeter) - 内网的后台服务

#通过micro api 进行HTTP请求，micro在逻辑上将API服务与后端服务分离。
#运行go.micro.api.greeter API服务，（这个的作用没搞清楚，而且还要专门为API写一个转发脚本，感觉繁琐。
之前以为一定要这个脚本再能进行HTTP访问。。。）
go run ./api/api.go

#运行micro api
micro api --handler=api （micro --registry=consul --registry_address=192.168.1.76:8500 api --handler=api）
#通过api访问go.micro.api.greeter,go.micro.api.greeter内部通过相应方法访问go.micro.srv.greeter对应方法
curl http://localhost:8080/greeter/say/hello?name=John


# Micro Web是一个Web仪表板和反向代理，用于将Web应用程序作为微服务运行
#运行go.micro.web.greeter服务
go run web/web.go
#运行micro web
micro web （micro --registry=consul web）
#原理同micro api
curl http://localhost:8082/greeter
```
##### PHP client
[官方教程](https://grpc.io/docs/quickstart/php)
```
/usr/local/bin/pecl install grpc
docker-php-ext-enable grpc
#php 项目 compose.yaml 添加
  "require": {
    "grpc/grpc": "v1.12.0"
  }

/usr/local/bin/pecl install protobuf
docker-php-ext-enable protobuf
#php 项目 compose.yaml 添加
  "require": {
    "google/protobuf": "^v3.5.0"
  }

//安装 grpc_php_plugin
git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc
cd grpc && git submodule update --init && make grpc_php_plugin

# protoc --php_out="protobuf/compile" "protobuf/protos/hello.proto"
protoc --proto_path=./protobuf/protos   --php_out=./protobuf/compile    --grpc_out=./protobuf/compile   --plugin=protoc-gen-grpc=/usr/local/grpc/bins/opt/grpc_php_plugin  ./protobuf/protos/helloworld.proto
```
```
<?php
/**
 * Created by PhpStorm.
 * User: Administrator
 * Date: 2019/5/28
 * Time: 18:09
 */

namespace App\Http\Controllers\Api\V1;

class ProtobufController
{
    public function send(){
        $service_name = 'go.micro.srv.greeter';
        $service = self::getService($service_name);
        $hostname = $service->Address.':'.$service->Port;
        $opts = [
            'credentials' => \Grpc\ChannelCredentials::createInsecure()
        ];

        $client = new \Hello\SayClient($hostname, $opts);
        $request = new \Hello\Request();
        $request->setName('hahahaha');
        list($reply, $status) = $client->Hello($request)->wait();
        echo $reply->getMsg();exit;

    }

    /**
     * @param string $service_name
     * @return object
     * @throws \Exception
     */
    public static function getService(string $service_name)
    {
        $consul_host = 'http://192.168.1.67:8500';

        //guzzle request options
        $options = [
            'base_uri' => $consul_host,
            'connect_timeout' => 3,
            'timeout' => 3,
        ];

        /**
         * @var \SensioLabs\Consul\Services\Health $h
         * @var \SensioLabs\Consul\ConsulResponse $resp
         */
        $h = (new \SensioLabs\Consul\ServiceFactory($options))->get('health');
        $resp = $h->service($service_name);
        $services = \json_decode($resp->getBody());
        if (!$services) {
            throw new \Exception('service '.$service_name.' not exists');
        }

        foreach ($services as $service) {
            if ($service->Service->Service != $service_name) {
                continue;
            }

            $del = false;
            foreach ($service->Checks as $check) {
                if ($check->Status == 'critical') {
                    $del = true;
                    break;
                }
            }
            if ($del) {
                continue;
            }
            //$host = $service->Service->Address.':'.$service->Service->Port;
            return $service->Service;
        }

        throw new \Exception('service '.$service_name.' not available');
    }
}
```

##### 错误处理
- 错误1
```
build command-line-arguments: cannot load github.com/ugorji/go/codec: ambiguous import: found github.com/ugorji/go/codec in multiple modules:
	github.com/ugorji/go v1.1.4 (/root/go/pkg/mod/github.com/ugorji/go@v1.1.4/codec)
	github.com/ugorji/go/codec v0.0.0-20190320090025-2dc34c0b8780 (/root/go/pkg/mod/github.com/ugorji/go/codec@v0.0.0-20190320090025-2dc34c0b8780)
```
报错的字面意思是有一个包多个地方引用但版本不一致。解决办法见：https://github.com/ugorji/go/issues/279。

需要运行以下命令解决问题:
go get github.com/ugorji/go@v1.1.2

- 错误2
安装consul 执行 make tools: 警告：检测到时钟错误。您的创建可能是不完整的,解决办法执行
```
 touch *
```
- 错误3 
报 too many colons in address错误 ，不支持IPV6,禁用IPV6即可

##### 其它

###### 日志跟踪和监控
- jaeger(耶格):一个是它兼容OpenTracing API,写起来简单方便，一个是UI相较于Zipkin的更加直观和丰富，还有一个则是sdk比较丰富，go语言编写，上传采用的是udp传输，效率高速度快。相比Pinpoint的缺点，当然是UI差距了，基本上现在流行的追踪系统UI上都远远逊于它。
```
docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 9411:9411 \
  jaegertracing/all-in-one:latest
```
- Zipkin(小精灵)

###### 服务监控
- Prometheus+grafana
    ###### prometheus介绍
    prometheus是由谷歌研发的一款开源的监控软件，目前已经被云计算本地基金会托管，是继k8s托管的第二个项目。
    prometheus采集到的监控数据均以metric（指标）形式保存在时序数据库中（TSDB）

    ###### Grafana介绍
    Grafana是一个跨平台的开源的度量分析和可视化工具，可以通过将采集的数据查询然后可视化的展示，并及时通知。它主要有以下六大特点：
    - 1、展示方式：快速灵活的客户端图表，面板插件有许多不同方式的可视化指标和日志，官方库中具有丰富的仪表盘插件，比如热图、折线图、图表等多种展示方式；
    - 2、数据源：Graphite，InfluxDB，OpenTSDB，Prometheus，Elasticsearch，CloudWatch和KairosDB等；
    - 3、通知提醒：以可视方式定义最重要指标的警报规则，Grafana将不断计算并发送通知，在数据达到阈值时通过Slack、PagerDuty等获得通知；
    - 4、混合展示：在同一图表中混合使用不同的数据源，可以基于每个查询指定数据源，甚至自定义数据源；
    - 5、注释：使用来自不同数据源的丰富事件注释图表，将鼠标悬停在事件上会显示完整的事件元数据和标记；
    - 6、过滤器：Ad-hoc过滤器允许动态创建新的键/值过滤器，这些过滤器会自动应用于使用该数据源的所有查询。
```
#Prometheus (普罗米修斯)
docker run -d --network=host -v /var/www/shopping-micro-srv/prometheus.yml:/etc/prometheus/prometheus.yml --name prometheus prom/prometheus
#访问 宿主机IP:9090

#grafana (格拉法纳)
docker run -d -p 3000:3000 --network go-micro-net --name grafana grafana/grafana 
#访问 host-ip:3000
```

###### 消息队列和分布式事务
- Rabbitmq

###### 服务降级和熔断
- Hystrix(好时tree)

##### 参考文档
- [官方中文文档](https://micro.mu/docs/cn/go-micro.html)
- [Laravel + go-micro + grpc 实践基于 Zipkin 的分布式链路追踪系统](https://mp.weixin.qq.com/s/JkLMNabnYbod-b4syMB3Hw)
- [基于Go Micro的微服务架构本地实战](https://www.codercto.com/a/30019.html)
- [micro微服务框架梳理](https://www.wandouip.com/t5i245217/)
- [全链路监控Jaeger搭建实战](https://www.jianshu.com/p/ffc597bb4ce8)
- [jaeger官方安装教程](https://www.jaegertracing.io/docs/1.12/getting-started/)
- [使用golang构建高可用微服务-概述](https://www.jianshu.com/p/91786e427939)
- [go-micro项目实战四 链路追踪](https://blog.csdn.net/u013705066/article/details/89530788)
- [Go 微服务，第11部分：Hystrix和Resilience](https://cloud.tencent.com/developer/article/1157926)
- [如何快速部署 Prometheus](https://www.cnblogs.com/CloudMan6/p/7724576.html)
- [go-micro In Action](https://cloud.tencent.com/developer/article/1456489)
- [Golang微服务开发实践](https://juejin.im/post/5cfa1b5b6fb9a07ecf721696#heading-1)
- [微服务架构最佳实践](https://juejin.im/post/5cbbe051f265da03973aabcb#heading-24)
- [从零搭建Prometheus监控报警系统](https://www.cnblogs.com/chenqionghe/p/10494868.html)
- [使用golang编写prometheus metrics exporter](https://yq.aliyun.com/articles/683180)
- [superbalist/laravel-prometheus-exporter](https://packagist.org/packages/superbalist/laravel-prometheus-exporter)
- [浅谈架构是为了什么](https://github.com/CrazyCodes/Blog/blob/master/浅谈架构是为了什么%20(下).md)
- [微服务架构下分布式事务解决方案——阿里GTS](https://www.cnblogs.com/jiangyu666/p/8522547.html)
- [Prometheus 介绍](https://www.cnblogs.com/zqj-blog/p/10871033.html)
- [实例讲解Springboot整合OpenTracing分布式链路追踪系统（Jaeger和Zipkin)](https://www.cnblogs.com/larrydpk/p/12638344.html)
