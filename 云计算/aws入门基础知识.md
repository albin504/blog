# IAM
即Identity and Access Management。身份及访问管理。
用来“管理对AWS资源的访问”。

## 场景1: 一家初创公司使用aws云服务，要给某些员工进行授权。
操作步骤：
1、公司在aws注册一个根账户（或者叫主账户）。

2、主账户可以给每个员工开通一个子账户（即IAM账户），并给子账户进行授权（如给开发组长授权运维S3服务）
<img width="420" alt="Pasted Graphic 2" src="https://user-images.githubusercontent.com/3232275/219934991-68c2be61-72a8-4538-a229-e21514416f89.png">

## 场景2: A业务的文件类数据，存储在S3服务上，现在要给B业务开放部分数据的查看权限。这里要解决一个问题：在api层面，如何识别用户的身份？
aws的做法是：
1、给业务B开通一个IAM账户。

2、在IAM账户中，添加access key(访问密匙。 https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html?icmpid=docs_iam_console#Using_CreateAccessKey)


