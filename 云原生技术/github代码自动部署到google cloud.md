# 项目说明

以往代码部署上线，我们需要手动执行一系列操作：
1. 代码编译。如java打jar包、php执行composer install下载依赖等。
2. 打docker镜像、并把镜像推送到中央仓库
3. 登录到服务器上更新代码（更新镜像）
4. 重启服务。
5. ......

如果是多实例部署，上述操作要重复多遍。

当然如果你在一家大公司供职，上述部署一般是通过公司的持续集成、持续交付系统自动化完成的。

如果是作为个人开发者，在服务部署这个环节，怎样才能做到自动化呢？

我基于谷歌翻译api，使用go 语言，开发了一个支持中英翻译的api服务。

    代码库地址：https://github.com/albin504/google-translation-api
    
<img width="756" alt="image" src="https://user-images.githubusercontent.com/3232275/222763258-bdee9cd2-ce82-4983-944e-b12e5fa37242.png">


本文会介绍该项目的自动化部署效果：**当我把代码提交到github仓库，大约2分钟时间，更新就自动上线了。**

它依赖的服务有：
1. github管理代码仓库
2. docker镜像存储在谷歌云的中央仓库
3. 容器运行在谷歌云的容器托管平台（cloud run），该平台支持容器多实例部署，部署简单、服务可靠。


# 自动化部署配置方法

在这里，自动化部署是基于github action完成的。

github action 可以简单理解成是一个自动化部署脚本。 当代码库代码发生变化，会触发github action的执行。自动化部署的关键步骤如下：
1. Authenticate to Google Cloud
2. Authenticate Docker to Artifact Registry
3. Build a docker container
4. Publish it to Google Artifact Registry
5. Deploy it to Cloud Run

总结起来，分三步： 
1. github和谷歌云之间进行授权
2. build docker镜像并推送到谷歌云中心仓库
3. 把镜像部署到谷歌云容器托管平台

这样，你的新版程序就完成了发布。

下面介绍其中的知识点：

## 授权
通过github action，要往谷歌云提交docker镜像、并把镜像部署到谷歌云容器托管平台。

这个过程需要得到谷歌云的授权。

我们应该采用哪种授权方法呢？

方式一： 在github上输入谷歌云的输入帐号、密码。 有密码泄漏的风险

方式二：谷歌云给你的帐号分配一个secert key，这个密钥可以代表你的身份，然后在github平台使用这个密钥就可以调用谷歌云的接口了。

这种方式免去了密码泄漏的风险。

方式三：基于oauth2.0协议。 微信公众平台授权就是基于该协议，当第三方开发者想要获取微信用户的身份信息（ID、头像、昵称）等信息时，第三方开发者和微信之间和协商一个临时的access token，用这个token就可以进行可信任交互了。

它的特点是，secert key也不需要了，就免去了secert key泄漏的风险。

方式四：OIDC(OpenID Connect)。 它和方式三很像，我也没具体了解两者的差异

    https://docs.github.com/zh/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-google-cloud-platform

github action就是基于OIDC协议和谷歌云进行授权的。

## 服务帐号

提到授权，一定绕不开谷歌云的服务帐号的概念，我花了很多时间才搞明白它。

服务帐号解决服务之间授权的问题。如，A服务要访问B服务，就要先得到B的授权。

我前面部署的谷歌翻译api，容器托管平台需要调用谷歌翻译的api。因此，我们假定这里的A服务就是谷歌云上的云托管平台（cloud run），B是谷歌翻译api服务。谷歌翻译api是要收费的，因此不能允许任何人随意调用，要校验调用方的身份。

谷歌云是这样做的：

1、 给容器托管平台cloud run分配一个服务帐号。服务帐号就代表了它的身份。
    https://cloud.google.com/compute/docs/access/service-accounts#default_service_account

官方文档有说明，cloud run服务的默认服务帐号是：PROJECT_NUMBER-compute@developer.gserviceaccount.com

2、给服务帐号分配一个“谷歌翻译api管理员”的角色。此时，该服务帐号就可以访问谷歌翻译api了，即cloud run上运行的容器（即我的go代码）无需密钥就可以调用google translation api了。

配置方式如下：

<img width="1664" alt="image" src="https://user-images.githubusercontent.com/3232275/222778484-e7a99757-932e-465d-8a36-5d0ba5e1688e.png">

先找到服务帐号，然后给它增加一个“Cloud Translation API Admin” 的角色，就可以了。

配置完成之后，看到服务帐号中已经关联该角色了：
<img width="1499" alt="Pasted Graphic" src="https://user-images.githubusercontent.com/3232275/222778079-e4fa7328-4680-48b9-84bd-4dbfb6bd27aa.png">

## 自动化部署中各服务授权关系链

1. gihub action 要访问镜像中心仓库、执行cloud run部署操作，需要通过服务帐号进行授权。

配置方式： 找到Artifact Registry 服务，给服务帐号（github-service-user）授予此资源（也就是当前的使用的镜像仓库）的管理权限。也就是说，github-service-user这个用户就有权限往中心仓库
push镜像了，github-service-user代表了github action的身份，即gihub action可以push镜像了。

<img width="1694" alt="image" src="https://user-images.githubusercontent.com/3232275/222943921-c45601ca-6e36-4d10-a637-878876990cbc.png">



2. cloud run上调用谷歌翻译api，需要通过服务帐号进行授权。

这一步的配置方式，在前面已经有介绍。

**至此，已经完成了权限的配置。**

------





