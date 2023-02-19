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

绑定之后，会存储在本地文件** ～/.aws/credentials ** 中，后续再调用api，就不用再输入身份信息了。

# ARN
Amazon Resource Names (ARNs)。即亚马逊资源名字。ARN为每一个资源定义了一个唯一的资源ID
aws上任意一个实体，都可以称为“资源”。如
- arn:aws:iam::324325127702:user/s3_user 是IAM用户s3_user的资源ID
- arn:aws:s3:ap-southeast-1:324325127702:accesspoint/albin-s3-accesspoint   是s3接入点albin-s3-accesspoint的资源ID
每个资源都有了唯一的ID，这样服务之间相互访问时，只要提供资源ID就可以了。举例：
- 要给IAM用户s3_user开放s3服务只读权限，资源ID就可以代表这个用户。
- 通过api访问我的s3服务，通过“arn:aws:s3:ap-southeast-1:324325127702:accesspoint/albin-s3-accesspoint” 就定义了服务的ID。如果没有这个arn，调用者api中就需要设置很多参数（如区域、根帐号ID等）来告诉aws具体是要调用哪个服务。


# VPC
官方解释：

    Amazon Virtual Private Cloud (Amazon VPC) enables you to launch AWS resources into a virtual network that you've defined. 
    This virtual network closely resembles a traditional network that you'd operate in your own data center, with the benefits of using the scalable infrastructure of AWS.
    亚马逊虚拟私有云（Amazon VPC）使您能够将AWS资源启动到您定义的虚拟网络中。这种虚拟网络与您在自己的数据中心运行的传统网络非常相似，其优点是使用AWS的可扩展基础架构。
    
个人理解：VPC就像是公司内网。每家公司都有一个大的私有网络，就类似于VPC。

<img width="1379" alt="image" src="https://user-images.githubusercontent.com/3232275/219937772-94ac3dbf-5981-4ee7-9d1b-8679cba552a4.png">


## VPC的安全策略
如VPC B想访问VPC A中的redis服务，就可以在安全组中设置VPC A的流量入口规则：
<img width="1681" alt="image" src="https://user-images.githubusercontent.com/3232275/219937518-70d11bf5-68ef-4788-9a44-b5055b449ebe.png">
上图的设置，意思是允许vpc：031afe41a08e6f3dc 访问我的所有端口。
在aws上，如果在两个区域（举例，美国、新加坡）分别部署了不同的服务：
- 美国部署了数据库服务
- 新加坡部署了ec2
ec2和db，记得默认应该归属于不同的vpc，不允许他们之间进行相互访问。如果要允许访问，就要进行上面的安全设置。

## 子网
    A subnet is a range of IP addresses in your VPC. A subnet must reside in a single Availability Zone. After you add subnets, you can deploy AWS resources in your VPC.

一个vpc中可以创建多个子网。子网把vpc划分成多个网络，一个服务（如ec2）归属于某一个子网。
<img width="1233" alt="image" src="https://user-images.githubusercontent.com/3232275/219937994-04891af2-0b9e-4cdf-b8d3-e3c65ec27ff0.png">
<img width="1050" alt="image" src="https://user-images.githubusercontent.com/3232275/219937978-def9bb21-18ce-445e-9562-4ffffde8c917.png">

这个服务的ip是172.31.24.125，归属的子网网段是172.31.16.0/20。
网段后面的 **/20** 是什么意思呢？
意思ip地址的前20位是网络地址，后12位是主机地址，这个网段可以分配4095(2的12次方减1)个ip。这个网段的子网掩码是是255.255.15.0
<img width="556" alt="image" src="https://user-images.githubusercontent.com/3232275/219938754-6ec89e73-0982-4238-b16e-3cfc4a87b160.png">






