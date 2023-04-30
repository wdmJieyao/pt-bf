# PT-BF：qBittorrent 下载自动拆包工具试用教程（5.8之前有效）

### 配置条件：

- 公网服务器（至少1C1G）;
- JDK17环境;
- qBittorrent（推荐4.3.9版本以上，但配置缓存时需小心内存溢出）;
- VT或其他能配置分类的刷流工具（需要用的种子分类）。



### 拆包工具介绍:

内测阶段，暂时没有构建docker镜像，后期会使用docker一键拉取镜像配置部署的模式。目前只能提供编译后的jar包直接在虚拟机上运行。

- PT-BF-0.0.1-SNAPSHOT.jar （PT-BF程序）
- pt-bf-config.yml  （PT-BF 配置文件）



### 准备阶段

- 服务器安装JDK17环境

以Debian10/11为例，按顺序执行以下命令：

更新apt

`apt upgrade`

安装openjdk17

`apt install openjdk-17-jre openjdk-17-jdk`

- 新建PT-BF配置文件（文件名：pt-bf-config.yml），并配置qBittorrent和拆包相关信息

```
pt-bf:
  secretKey: ABCDashjkdhjaskhj
  #待拆包的种子对应的类型
  category: PT-BF
  #拆包后文件大小 按字节计算（50GB = 50*1024*1024*1024 = 53687091200）
  bfFileSize: 53687091200
  #过滤文件时的区间  单个文件最大50GB 按字节计算
  bfFilterFileMaxSize: 53687091200
  #过滤文件时的区间  单个文件最小10MB 按字节计算
  bfFilterFileMinSize: 10485760
  #定时任务表达式(默认1分钟1次)
  taskCron: 0 */1 * * * ?
  qbInfoList:
    #暂不支持https,尽量使用IP+端口的配置方式。
    - host: 88.222.222.222
      port: 8080
      username: ptbfusername
      password: ptbfpassword
    #暂不支持https,尽量使用IP+端口的配置方式。
    - host: 104.222.222.222
      port: 8080
      username: ptbfusername
      password: ptbfpassword
  
```

- 在刷流工具中配置待拆包的种子类型（推荐VT）

配置RSS任务时，一定要选择添加时暂停种子，种子分类为配置文件的category相对应即可。

![VT](https://lijieyao-blog.oss-cn-shenzhen.aliyuncs.com/image-20230430141706179.png)

### 服务器部署

创建配置文件目录(固定目录，一定要这个)

`mkdir -p /root/apps/PT-BF/config/`

进入到配置文件目录中，并上传YML配置文件

` cd /root/apps/PT-BF/config/`

![](https://lijieyao-blog.oss-cn-shenzhen.aliyuncs.com/image-20230430142119873.png)

上传程序主题jar包，并执行jar包

`cd`

`nohup java -jar -Dfile.encoding=utf-8 -Dspring.profiles.active=test PT-BF-0.0.1-SNAPSHOT.jar  >/tmp/null 2>&1 &`

查看日志

`tail -f /root/apps/PT-BF/logs/info.log`

最终效果

![](https://lijieyao-blog.oss-cn-shenzhen.aliyuncs.com/image-20230430142332519.png)

拆包前

![拆包前](https://lijieyao-blog.oss-cn-shenzhen.aliyuncs.com/96790d71de15963dc09938870d5e90f.png)

拆包后

![拆包后](https://lijieyao-blog.oss-cn-shenzhen.aliyuncs.com/af23ea4ca2e224a868b6b95a52113b9.png)
