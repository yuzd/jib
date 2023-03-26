### 工具背景

![image](https://dimg04.c-ctrip.com/images/0v51b12000at8kldg0BE7.png)

Google开发的jib**不依赖docker环境**也能**创建**docker或者OCI类型的**镜像**，但是可惜它只为java应用而生，其他类型的比如nodejs,.net应用都无法用，而且它是作为maven/gradle的插件形式来工作的，而不是一个纯粹独立构建镜像的工具。

介于上述原因，来介绍我开发的这款工具，名字也叫jib，只不过它是一个纯粹构建镜像工具，支持win,linux,osx三个平台

功能包含：
- 构建镜像推送到镜像仓库(dockerhub/aliyun/tencent共有仓库,harbor等私有仓库)
- 构建镜像推送到本地docker环境
- 构建镜像生成tar格式镜像文件到本地

工具地址： [https://github.com/yuzd/jib](https://github.com/yuzd/jib)


#### 工具使用
它是一个纯粹构建镜像命令行工具,根据不同的功能有不同的参数，如下图

###### windows平台
![image](https://dimg04.c-ctrip.com/images/0v54m12000at8klbiB9E2.png)

###### macos平台
![image](https://dimg04.c-ctrip.com/images/0v54612000at8mbjb8108.png)



作为一个纯粹的构建镜像工具，它不需要依赖docker环境,只需要读取一个json配置文件，根据配置生成镜像

#### json配置文件

```bash
命令： jib.exe -push --configfile=demo.json
```


推送到镜像仓库的配置示例(从阿里云镜像仓库拉取base镜像+我要加的目录=新的镜像并推送到私有仓库)：
```json

{
  "BaseHttpProxy": "",
  "BaseImage": "ccr.ccs.tencentyun.com/dotnet-core/aspnet:2.2",
  "BaseImageCredential": {
    "UserName": "aaaaaaaa",
    "Password": "xxxxx"
  },
  "TargetHttpProxy": "",
  "TargetImage": "http://127.0.0.1:5000/test1",
  "TargetTags": [
    "1.0.1"
  ],
  "TargetImageCredential": {
    "UserName": "aaaa",
    "Password": "xxxx"
  },
  "ImageFormat": "Docker",
  "ImageLayersFolder": "E:\\workspace\\demo\\publish",
  "ImageWorkingDirectory": "/publish",
  "Entrypoint": [
    "dotnet"
  ],
  "Cmd": [
    "/publish/RazorTestProject.dll"
  ],
  "ApplicationLayersCacheDirectory": "E:\\workspace\\cache",
  "Env":{
      "env1":"value1"
  },
  "Ports":[
    {
        "port":8080,
        "protocol":"tcp"
    }
  ],
  "Volumes":[
    "/var/log",
    "/var/log2"
  ]
}

```



| 字段名 | 含义 | 备注 |
| --- | --- | --- |
| BaseHttpProxy | 代理 | 拉取基础镜像的时候看你需要，格式 ip:port  |
| BaseImage | 基础镜像地址 | 完整地址，包含了版本，如果仓库地址没有https，请在最前面加上http://  |
| BaseImageCredential | 拉取基础镜像如果要登录 | 账户名+密码  |
|TargetHttpProxy  | 代理 | 只有在推送到远程镜像且你有需要，才需要配置 格式ip:port |
| TargetImage | 目标镜像 | 要推送的目标镜像仓库地址，不包含版本，如果仓库地址没有https，请在最前面加上http:// |
| TargetTags | 镜像标签 | 可以理解为版本号  |
|TargetImageCredential  |  如果目标镜像仓库要登录| 账户名+密码 |
|ImageFormat  |  镜像仓库构建格式| Docker和OCI两种 |
|ImageLayersFolder  |  要打包进镜像仓库的目录| 通常这就是你的项目成果物 |
|ImageWorkingDirectory  |  打包的目标仓库的工作目录| 如果设置那你的文件们都会在这个目录下工作 |
|Entrypoint  |  镜像启动的入口| 比如dotnet |
|Cmd  |  镜像启动执行的参数| 供Entrypoint使用 |
|ApplicationLayersCacheDirectory  |  程序在运行时候会产生缓存目录来加快下次构建速度| 可以不指定,会用temp目录 |
|Env  | 环境变量 | 可以不指定,容器启动指定也行 |
|Ports  | 端口 | 可以不指定,容器启动指定也行 |
|Volumes  | 共享目录 | 可以不指定,容器启动指定也行 |


#### tar格式镜像文件本地生成

```bash
命令： jib.exe -tar --configfile=demo.json --outfile=demo.tar
```

示例
```json

{
  "BaseHttpProxy": "",
  "BaseImage": "ccr.ccs.tencentyun.com/dotnet-core/aspnet:2.2",
  "BaseImageCredential": {
    "UserName": "aaaaaaaa",
    "Password": "xxxxx"
  },
  "ImageFormat": "Docker",
  "ImageLayersFolder": "E:\\workspace\\demo\\publish",
  "ImageWorkingDirectory": "/publish",
  "Entrypoint": [
    "dotnet"
  ],
  "Cmd": [
    "/publish/RazorTestProject.dll"
  ]
}

```

json配置参数就少了推送相关的参数

本地tar文件的镜像，可以通过docker load命令在装载到docker环境中。

#### 推送镜像到本机的docker环境


```bash
命令： jib.exe -deamon --configfile=demo.json
```

和tar差不多

