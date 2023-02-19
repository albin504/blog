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

2、在IAM账户中，添加access key(访问密钥。 https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html?icmpid=docs_iam_console#Using_CreateAccessKey)

访问密钥由access key 和secret key构成。 access key用来指代用户是谁，secret key是密钥，不能公开（只有用户和AWS服务器知道密钥）。
发送api请求时，一般会用secret key对url进行加密，aws收到请求后，会解密判断密钥是否正确，防止其他用户伪造请求。

最近大火的chatgpt,用户必须注册账户才能使用。同时，chatgpt也开放了api，第三方开发者可以通过api，调用机器人聊天服务。
<img width="1391" alt="image" src="https://user-images.githubusercontent.com/3232275/219935726-5c77b5fb-e805-4add-ba06-1ad1e0b42218.png">
如上图，通过chrome浏览器插件，可以实现“当在搜索引擎进行搜索时，同时出现chatgpt的结果”。
使用这个插件，必须要先绑定用户的chatgpt账号。具体怎么绑定呢？
<img width="637" alt="image" src="https://user-images.githubusercontent.com/3232275/219935828-d58f8459-b941-4b76-a337-6e30f7040e08.png">
chatgpt官方给每个帐号生成了apikey，在chrome插件中填写api key，就完成了帐号绑定。

这里的原理和IAM中的访问密钥是一个思路。用户调用api，为了识别用户身份，我们不能要求用户填写帐号密码，所以就用**访问密钥**来指代身份。

3、业务B，通过access key 和secret key，就可以调用S3的api获取业务A的文件数据了。
aws cli是一个很好用的工具。
比如我们要从s3上下载一个文件。用aws cli命令如下：
```
aws s3api get-object --key 2022Q4_alphabet_earnings_release.pdf --bucket albin-accesspoint-2-8u7tq8jfy3k53bfun1auiiurhnpuraps1a-s3alias  1.pdf
```
返回结果：
```
{
    "AcceptRanges": "bytes",
    "LastModified": "2023-02-19T04:58:21+00:00",
    "ContentLength": 918154,
    "ETag": "\"f40b5ea1ff1db9a7eea43474500fca60\"",
    "VersionId": "__OylMawrp1tg6_yZlCAqaNkek2geVW1",
    "ContentDisposition": "attachment",
    "ContentType": "application/octet-stream",
    "ServerSideEncryption": "AES256",
    "Metadata": {}
}
```
太方便不过了。
使用aws cli之前，需要先绑定用户。（如下面，输入access key、secret key，就完成了身份绑定）
<img width="520" alt="image" src="https://user-images.githubusercontent.com/3232275/219936110-d26bb1c4-27d1-47d8-b11f-960aba4ec94c.png">
