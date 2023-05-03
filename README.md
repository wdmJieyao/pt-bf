# PT-BF：qBittorrent 下载自动拆包工具试用教程（看心情更新）

### 版本迭代

- 2023-05-03

1.优化拆包归并膨胀过大问题；

2.新增拆包比例设置，目前可以和拆包后固定大小配合，两者比较取小进行拆包；

3.优化系统鉴权逻辑，减少系统卡顿。



### 配置条件：

- 公网服务器（至少1C1G）;
- docker
- qBittorrent（推荐4.3.9版本以上，但配置缓存时需小心内存溢出）;
- VT或其他能配置分类的刷流工具（需要用的种子分类）。

### 准备阶段

- 服务器安装docker

以Debian 10为例，按顺序执行以下命令：

更新apt

`apt upgrade`

安装docker

` curl -fsSL https://get.docker.com -o get-docker.sh`

` sudo sh get-docker.sh`

检查环境

```
root@localhost:/data/PT-BF# docker -v
Docker version 23.0.5, build bc4487a
```

有docker版本即为成功

- 新建PT-BF配置文件（文件名：pt-bf-config.yml），并配置qBittorrent和拆包相关信息

```
pt-bf:
  secretKey: d7789010-166d-4c14-a199-f4a3dc6308c5
  #全局拆包分类，不配置扫描整个QB的所有种子
  # category: PT-BF
  #拆包后文件大小 按字节计算（50GB = 50*1024*1024*1024 = 53687091200）
  bfFileSize: 53687091200
  #过滤文件时的区间  单个文件最大50GB 按字节计算
  bfFilterFileMaxSize: 53687091200
  #过滤文件时的区间  单个文件最小10MB 按字节计算
  bfFilterFileMinSize: 10485760
  #定时任务表达式(默认1分钟1次)
  taskCron: 0 */1 * * * ?
  #种子状态筛选器，all-全数据扫描 默认为 paused 暂停
  torrentStateFilter: all
  # torrentStateFilter配置项为all时，过滤那些类型的种子拆包
  BFStateFilter: \,forcedDL,checkingDL,stalledDL,queuedDL,pausedDL,metaDL,downloading,\
  #触发拆包的种子大小 按字节计算
  needBFSize: 10485760
  #拆包后文件最小大小比率  50*0.4=20GB 若拆包后小于改值触发补偿机制，直至拆包文件大于最小值 
  needBFSizeMinRatio: 0.4
  #拆包后文件最大大小比率 50*(1+0.4) = 70GB 若拆包后大于该值则触发补偿机制，直至拆包文件小于最大值
  needBFSizeMaxRatio: 0.1
  #拆包比例 拆包后文件大小 = 原始文件大小* 拆包比例
  bfFileOriginalRatio: 0.3
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

参数说明：

secretKey: d7789010-166d-4c14-a199-f4a3dc6308c5

目前为临时key，最大能绑定50台QB，并且有效时间。

- 上传配置文件

创建配置文件目录（固定且唯一）

`mkdir -p mkdir -p /data/PT-BF/config`

进入配置文件目录

`cd /data/PT-BF/config`

上传配置文件,且名称一定要为pt-bf-config.yml



#### 单机、纯小鸡配置用分类拆包更优

- 在刷流工具中配置待拆包的种子类型（推荐VT）

配置RSS任务时，一定要选择添加时暂停种子，种子分类为配置文件的category相对应即可。

![VT](https://lijieyao-blog.oss-cn-shenzhen.aliyuncs.com/image-20230430141706179.png)

### 服务器部署

1.拉取镜像,注意替换版本号，目前最新为0.0.3版本。

`docker pull wdmjieyao/pt-bf:vPT_BF_0.03`

2.使用一下命令启动

`docker run -d -p 7758:7758 -v /data/PT-BF/config:/data/PT-BF/config --name PT-BF --restart=always wdmjieyao/pt-bf:vPT_BF_0.03`

3.查看镜像日志

`docker logs -f PT-BF`

最终效果

![](https://lijieyao-blog.oss-cn-shenzhen.aliyuncs.com/image-20230430142332519.png)

拆包前

![拆包前](https://lijieyao-blog.oss-cn-shenzhen.aliyuncs.com/96790d71de15963dc09938870d5e90f.png)

拆包后

![拆包后](https://lijieyao-blog.oss-cn-shenzhen.aliyuncs.com/af23ea4ca2e224a868b6b95a52113b9.png)

### 未来的计划

- ~~docker一键部署~~（已完成）

- 1.拆包比例设置。

- 2.根据时间段调整拆包比例，比如深夜时段不太适合拆的太小。

- 3.拆包策略 首先分析出这个包是电影 还是剧集大包，电影得话 如果包超过某个大小就丢掉，剧集得话建议按比例拆，拆完后如果包还是很大，分析是否还有可拆空间，进行二次拆。

- ...

  
