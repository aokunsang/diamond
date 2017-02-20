diamond
=======

来自淘宝diamond：http://code.taobao.org/p/diamond/src/

一、diamond-server
 
 一个web程序，用tomcat容器启动。  
1.提供接口用于客户端获取配置信息，以及探测配置信息是否修改并且获取最新数据，  
2.提供一个简约的操作页面，增删改配置数据，  
3.通知集群中其他节点同步更新变化的配置数据，集群节点的ip和port，通过node.properties配置(这个不太好，如果在集群中新增一个节点，需要更改其他所有节点配置)，  
4.system.properties中配置的dump_config_interval，定时器多久全量更新一次数据库的配置数据，  
  说明：一次取1000条记录，分页更新。

二、客户端配置  
客户端调用diamond-server获取配置数据信息时，需要通过diamond.properties动态修改一些数据，如：

//获取ServerAddress的URI服务器所在域名或者ip地址  
DEFAULT_DOMAINNAME=localhost

//获取ServerAddress的URI备用服务器所在域名或者ip地址  
DAILY_DOMAINNAME=localhost

//获取ServerAddress的URI服务器的端口号(nginx为80)  
DEFAULT_PORT=8080

//获取配置数据的URI地址(可以是http全路径地址)，如果不带ip，那么轮换使用ServerAddress中的地址请求  
HTTP_URI_FILE=/diamond-server/config.co

//获取ServerAddress的URI(可以是http全路径地址)，diamond是一个文件，一行一个ip或者域名
CONFIG_HTTP_URI_FILE=/diamond-server/diamond  

//探测某个配置数据的时间间隔，默认为15秒  
POLLING_INTERVAL_TIME=1  

说明：  
1、如果HTTP_URI_FILE和CONFIG_HTTP_URI_FILE的端口号不一样，需要修改源代码设置。  
HTTP_URI_FILE对应的port修改位置：com.taobao.diamond.client.DiamondConfigure#port  
CONFIG_HTTP_URI_FILE的port修改位置：com.taobao.diamond.common.Constants#DEFAULT_PORT，并且可以动态配置，参考上面；  

2、HTTP_URI_FILE和CONFIG_HTTP_URI_FILE的ip地址获取方式：  
CONFIG_HTTP_URI_FILE，先取com.taobao.diamond.client.DiamondConfigure#configServerAddress，  
(1)如果configServerAddress!=null(不会重试备用服务器，只会重试2次同样的host+port，推荐默认值null)，host=com.taobao.diamond.client.DiamondConfigure#configServerAddress，port=om.taobao.diamond.client.DiamondConfigure#configServerPort;  
(2)如果configServerAddress==null(会重试备用服务器)，
host=com.taobao.diamond.common.Constants#DEFAULT_DOMAINNAME，port=com.taobao.diamond.common.Constants#DEFAULT_PORT;
重试备用服务器时，host=com.taobao.diamond.common.Constants#DAILY_DOMAINNAME，port=com.taobao.diamond.common.Constants#DEFAULT_PORT;


三、获取diamond-server服务器的地址  
1、客户端启动时先从CONFIG_HTTP_URI_FILE取一次地址列表并且写入本地文件(~/diamond/ServerAddress，默认会写入一个localhost)，然后放入DiamondConfigure.domainNameList中,异步线程每隔5分钟从CONFIG_HTTP_URI_FILE获取有效的地址，更新本地文件。   
2、客户端本地文件存储目录：~/diamond/data，项目启动时同时注册文件夹(增加、删除、修改)监控事件。
