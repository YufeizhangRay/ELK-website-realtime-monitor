# ELK-website-realtime-monitor  
  
## Elasticsearch + Logstash + Kibana + Nginx 搭建网站实时监控平台  
  
### 架构介绍  
  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/ELKintro.jpeg)  
  
通过上图我们可以看到，ELK 是由三个 Elastic 的产品组合而成，分别是 ElasticSearch、Logstash 和 Kibana。三者之间的部署关系如下图所示:  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/ELKrelation.jpeg)  
  
Logstash 就好比是挖矿工，将原料采集回来存放到 ElasticSearch 这个仓库中，Kibana 再将存放在 ElasticSearch 中的原料 进行加工包装成产品，输出到 web 界面。基本工作原理如下图所示  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/archtect.jpeg)  
  
### 1.Elasticsearch分布式环境搭建  
#### 1.1 下载  
下载地址：https://www.elastic.co/downloads/past-releases (建议下载统一的ELK版本以确保最高的稳定性，此时的最新版本6.5.1)  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/downloades.jpeg)  
  
下载完毕后直接解压即可。  
  
#### 1.2 参数配置  
找到config文件夹中的jvm.options文件，我们可以设置参数  
 -Xms4g  
 -Xmx4g  
 ![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/JVM.jpeg)  
   
Elasticsearch非常的吃内存，我们可以将这两个参数设置的大一些。  
  
#### 1.3 分布式环境准备  
将下载好的Elasticsearch拷贝为一共三份(为集群)，一个作为master节点，两个作为slave节点(如上图)  
修改主节点的配置，打开 elasticsearch-6.5.1-master\config 下的 elasticsearch.yml 文件，在底部追加如下内容:  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/master.jpeg)  
  
配置 slave-1 节点，打开 elasticsearch-6.5.1-slave-1\config 下的 elasticsearch.yml 文件，在底部追加如下内容:  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/slave1.jpeg)  
  
配置 slave-2 节点，打开 elasticsearch-5.5.1-slave-2\config 下的 elasticsearch.yml 文件，在底部追加如下内容:  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/slave2.jpeg)  
  
使用 ./elasticsearch分别启动三个节点。    
  
### 2.Logstash
#### 2.1 下载并启动  
官网 https://www.elastic.co/cn/downloads/past-releases 下载 Logstash 6.5.1,解压即可。  
启动方式一:  
命令行输入: bin\logstash -e 'input { stdin {} } output { stdout {} }'  
启动方式二:在 config 目录下新建 logstash.conf 文件，编辑以下内容:  
input {  
    stdin {}  
}  
output {  
    stdout {}   
}  
控制台输入以下命令:  
bin\logstash -f config\logstash.conf  

#### 2.2 访问日志生产平台的搭建  
为让演示效果更加真实，这里直接利用 Nginx 产生的访问日志作为流量监控的元数据。因此，自己要先搭建 Nginx 运行环境，并部署一个可以访问的 web 项目。然后，在 logstash 的安装目录新建一个 patterns 目录，在此目录下创建 nginx 空白文件，内容如下:  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/Nginx.jpeg)  
  
最后，对 logstash.conf 中的内容进行修改:  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/logstashconf.jpeg)  
  
启动 logstash 便可以将 Nginx 日志同步到 logstash 中来。  

#### 2.3 Logstash 与 ElasticSearch 集成  
在 logstash.conf 追加以下内容(上图已包含)，即可与 Elasticsearch 实现无缝集成:  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/logstache's.jpeg)  
  
若想要Logstash也被Kibana监控到，需要在logstash.yml中加入一项配置：  
xpack.monitoring.elasticsearch.url: "http://127.0.0.1:9200"  
  
### 3.利用 Kibana 实现网站流量可视化   
#### 3.1 可视化插件安装  
下载 NodeJS 环境，打开官网 https://nodejs.org/en/download/  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/NodeJS.jpeg)  
  
根据系统选择相应版本，mac下载后直接安装即可。可以输入 node -v 检查 node 是否安装成功。  
  
#### 3.2 下载 Kibana  
官网 https://www.elastic.co/cn/downloads/past-releases 下载 Kibana6.5.1,解压即可。  
命令行输入 ./kibana 即可启动，然后在浏览器输入 http://localhost:5601 即可看到可视化界面:   
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/KibanaUI.jpeg)  
  
至此，ELK的整合完毕。

### 4.配置并启动Nginx  
在conf文件夹下创建vhost文件夹并在其中创建配置文件zyf.test.conf  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/zyftestconf.jpeg)  
  
内部配置如下  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/Nginxconf.jpeg)  
  
手动选择配置文件启动，可以使用如下指令:  
./nginx -c /Users/ray_cn/framework/nginx-1.12.2/nginx/conf/vhost/zyf.test.conf  
  
### 5.实现网站实时监控  
随意启动一个web项目即可，这里使用之前做过的一个项目。  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/web.jpeg)  
  
由于端口号为9090，Nginx为我们做了代理服务器，并记录了访问日志。此时logstash会自动读取新的日志  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/readlog.jpeg)  
  
于此同时，Kibana的可视化界面也实时更新了监控数据(此过程暂时略)  
![](https://github.com/YufeizhangRay/image/blob/master/elasticsearch/Kibana.jpeg)

