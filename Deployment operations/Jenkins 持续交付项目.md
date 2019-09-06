### 首先在GitLab的项目上创建标签
![](https://www.funtl.com/assets/Lusifer_20181029033939.png)
![](https://www.funtl.com/assets/Lusifer_20181029034014.png)
### 创建 Maven Project
#### 在 Jenkins 中创建一个基于 Maven 的任务
#### 配置第一次构建
- 填写以下信息
```
# 项目名称
myshop-dependencies

# 描述
xxxxxxx

(√)丢弃旧的构建

# 源码管理 -> (√)Git
# URL
ssh://git@xxxxx.git
```
- 保存后,点击`立即构建`,就会自动拉取代码和构建项目.
- 第一次构建的目的主要目的是创建`Maven仓库目录`和`克隆代码`
#### 配置正式构建(只构建不部署)
- 在项目设置里,,填上以下参数
```
(√)参数化构建过程
# Extend Choise Parameter
# Name
RELEASE_VERSION

(√)Basic Parameter Types
(单选)Single Select

# Choose Source For Value
# Groovy Script -> 获取最近的5次Tags版本
def ver_keys = [ 'bash', '-c', 'cd /var/jenkins_home/workspace/myshop-dependencies;git pull>/dev/null; git remote prune origin >/dev/null; git tag -l|sort -r |head -10 ' ]
ver_keys.execute().text.tokenize('\n')

# 源码管理,由Git改为无

# Pre Steps 增加构建步骤
(√)执行shell
# Command
echo $RELEASE_VERSION
cd /var/jenkins_home/workspace/myshop-dependencies
git checkout $RELEASE_VERSION
git pull origin $RELEASE_VERSION
mvn clean package
```
- 注意`myshop-dependencies`是GitLab代码项目名称,每次需更换
- 填完保存,构建选项会变成`使用参数化构建项目`
- 第一次点击构建,会提示测试Groovy脚本正确性,点击`In-Process Script Approval`->`Approve` 测试后才可以选择`Tags` ,正常构建
```
Maven project myshop-service-user-consumerThis build requires parameters:You have unapproved groovy scripts that need approval before they can be executed. Go to `In-Process Script Approval` page to approve your scripts.
```

### 构建Dubbo服务提供者/消费者(部署)
#### 配置第一次构建(和上面一致)
#### 配置正式构建(再上面的基础上增加配置)
```
# Pre Steps 增加构建步骤
# 再增加构建步骤 Send files or excute commands over SSH
# Name 自动识别之前绑定的可控制设备

# Source Tree
**/*.jar,docker/**

# Remote directory
myshop-service-user-provider

# Exec command
cd /usr/local/jenkins/myshop-service-user-provider
cp target/myshop-service-user-provider-1.0.0-SNAPSHOT.jar docker
cd docker
docker build -t 192.168.229.137:5000/myshop-service-user-provider:v1.0.0 .
docker push 192.168.229.137:5000/myshop-service-user-provider:v1.0.0
docker-compose down
docker-compose up -d
docker image prune -f
```
- 注意: `192.168.229.137`是`Registry`的地址,部署目标主机需要提前配置好对应的Registry仓库，否则会报错:
```
The push refers to repository [192.168.229.137:5000/myshop-service-user-provider]
Get https://192.168.229.137:5000/v2/: http: server gave HTTP response to HTTPS client
```
- 还有手动创建Docker沙箱网络
```
docker network create dubbo
```
- 填完保存,构建选项会变成`使用参数化构建项目`