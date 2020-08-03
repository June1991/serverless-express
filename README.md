本文以tencent-express组件部署一个express网站为例，讲解serverless framework开发项目、管理项目和部署发布上线全流程。示例链接： https://github.com/June1991/serverless-express 

一个项目的开发上线流程大致如下:

![1593660643456](https://github.com/June1991/serverless-express/blob/master/doc/img/1593660643456.png)

初始化项目：项目进行初始化。比如选择一些开发框架和模板完成基本的搭建工作。

开发阶段：对产品功能进行研发。可能涉及到多个开发者协做，开发者拉取不同的feature分支，开发并测试自己负责的功能模块。最后合并到dev分支，联调各个功能模块。

测试阶段：测试人员对产品功能进行测试。

发布上线：对于已完成测试的产品功能发布上线。由于新上线的版本可能有不稳定的风险，所以一般会进行灰度发布，通过配置一些规则监控新版本的稳定性，等到版本稳定后，流量全部切换到新版本。



## 1.基本概念

#### 项目概念



一个serverless应用是由单个或者多个组件实例构成。

每个组件中都会有一个serverless.yml文件，该文件定义了组件的一些参数，这个些参数在部署时用于生成实例的信息。比如region，定义了资源的所在区。

组织是在serverless应用上层的概念，主要是为了管理。如一个公司会有不同部门进行serverless应用开发，设置不同组织名称，方便做后期的权限管理。

比如开发一个express应用，最基本的是引入express组件，业务中间可能会涉及到其他一些云产品比如cos，所以整个应用目录如下：

![1596187596773](https://github.com/June1991/serverless-express/blob/master/doc/img/1596187596773.png)



#### 分支概念

示例中将会涉及以下分支：

- master: 用于生产环境部署
- testing: 用于测试环境测试
- dev: 用于日常开发

- feature-xxx: 用于增加一个新功能，比如不同开发者会从dev拉取不同的特性分支进行开发
- hotfix-xxx: 用于修复一个紧急bug

#### 命令说明

##### 函数发布版本

```
sls deploy --inputs.publish="fun01,fun02" #部署时函数发项目下 fun01、fun02 的版本
sls deploy --inputs.publish  #部署时项目下所有函数发版本
```

##### 函数流量设置

```
sls deploy --inputs.traffic="0.2" #部署后切换20%流量到 $latest 版本
```

- Serverless Framework 流量切换修改的是云函数别名为 $default 的流量规则。
- traffic 配置的值为 $latest 版本对应的流量占比，最后一次云函数发布的版本的流量占比为 1-$latest 流量占比。例如，traffic="0.2"，实则配置 $default 的流量规则为 {$latest:0.2, 最后一次云函数发布的版本: 0.8}
- 如果函数还未发任何固定版本，只存在 $latest 版本的函数时，traffic 无论如何设置，都会是 $latest:1.0。
#### serverless framework参数

示例中会通过stage参数进行环境的隔离，不同的stage配置最后部署到不同的云函数上。

```
# serverless.yml

org: xxx-department #  用于记录组织信息,默认为您的腾讯云appid
app: expressDemoApp #  应用名称，默认为与组件实例名称
stage: ${env:STAGE} #  用于开发环境的隔离，默认为dev


component: express # (必填) 引用 component 的名称，当前用到的是 express-tencent 组件
name: expressDemo # (必填) 组件创建的实例名称


inputs:
  src:
    src: ./ 
    exclude:
      - .env
  region: ap-guangzhou 
  runtime: Nodejs10.15
  funcitonName: ${name}-${stage}-${app}-${org} #云函数名称
  apigatewayConf:
    protocols:
      - http
      - https
    environment: release
```



## 2.初始化项目

1、根据官网指引[部署 Express.js 应用]( https://cloud.tencent.com/document/product/1154/43224 )创建一个express项目，修改yml文件为：

```
#serverless.yml
org: xxx-department #  用于记录组织信息,默认为您的腾讯云appid
app: expressDemoApp #  应用名称，默认为与组件实例名称
stage: ${env:STAGE} #  用于开发环境的隔离，默认为dev


component: express # (必填) 引用 component 的名称，当前用到的是 express-tencent 组件
name: expressDemo # (必填) 组件创建的实例名称

inputs:
  src:
    src: ./ 
    exclude:
      - .env
  region: ap-guangzhou
  runtime: Nodejs10.15
  funcitonName: ${name}-${stage}-${app}-${org} #云函数名称
  apigatewayConf:
    protocols:
      - http
      - https
    environment: release
```

3、在项目根目录下的.env文件中配置：

```
TENCENT_SECRET_ID=xxxxxxxxxx #您账号的 SecretId
TENCENT_SECRET_KEY=xxxxxxxx #您账号的 SecretKey
STAGE=prod #STAGE为prod环境，也可以sls deploy --stage=prod 参数传递的方式设置
```

4、执行sls  deploy部署成功后，访问生成的url链接，效果如下：
<img src="https://github.com/June1991/serverless-express/blob/master/doc/img/1593676706604.png" alt="1593676706604" style="zoom:33%;" />


5、创建远程仓库https://github.com/June1991/serverless-express ，将项目代码提交到远程master分支。同时创建testing、dev。此时三个分支的代码在同一个版本上（假设为版本0）。

<img src="https://github.com/June1991/serverless-express/blob/master/doc/img/1593675966800.png" alt="1593675966800" style="zoom:40%;" />



## 3. 开发&测试

#### 背景

现在需要开发某个功能模块。假设需要有两位开发者：Tom、Jorge。两位开发者分别从dev（版本0）上创建特性分支为feature1、feature2进行研发。

<img src="https://github.com/June1991/serverless-express/blob/master/doc/img/1593680347852.png" alt="1593680347852" style="zoom:40%;" />

Tom开始开发feature1。在本示例中，为新增一个feature.html，里面写文案"This is a new feature 1."。

#### 开发

1、sls.js 文件中新增路由器配置：

```
// Routes
app.get(`/feature`, (req, res) => {
 res.sendFile(path.join(__dirname, 'feature.html'))
})
```

2、新增feature.html：

```
<!DOCTYPE html>
<html lang="en">
 <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Serverless Component - Express.js</title>
 </head>
 <body>
  <h1>
   This is a new feature 1.
  </h1>
 </body>
</html>
```

3、为了在开发过程中得到独立的运行和调试环境，需要在.env文件中设置自己的stage，例如Tom在serverless.yml的项目目录下配置.env如下：

```
TENCENT_SECRET_ID=xxxxxxxxxx
TENCENT_SECRET_KEY=xxxxxxxx
STAGE=feature1
```

4、执行sls  deploy部署成功后，返回显示如下：

```
region: ap-guangzhou
apigw:
  serviceId:   service-xxxxxx
  subDomain:   service-xxxxxx-123456789.gz.apigw.tencentcs.com
  environment: release
  url:         https://service-xxxxxx-123456789.gz.apigw.tencentcs.com/release/
scf:
  functionName: express-demo-feature1
  runtime:      Nodejs10.15
  namespace:    default
  lastVersion:  $LATEST
  traffic:      1

Full details: https://serverless.cloud.tencent.com/instances/expressDemoApp%3Afeature1%3AexpressDemo

10s » expressDemo » Success
```

5、访问生成的url:https://service-xxxxxx-123456789.gz.apigw.tencentcs.com/release/feature，效果如下：
<img src="https://github.com/June1991/serverless-express/blob/master/doc/img/1593763446846.png" alt="1593763446846" style="zoom:40%;" />

至此，Tom开发功能完成并自测通过。

假设同时，Jorge同时也完成自己的特性开发，并自测通过。在本示例中，为新增一个feature.html，里面写文案"This is a new feature 2."。

#### 联调

1、两人把各自feature分支的代码合并到dev分支。（可能会存在冲突需要人为解决）
<img src="https://github.com/June1991/serverless-express/blob/master/doc/img/1593680818016.png" alt="1593680818016" style="zoom:40%;" />

2、在dev进行联调。联调环境中的.env配置如下

```
TENCENT_SECRET_ID=xxxxxxxxxx
TENCENT_SECRET_KEY=xxxxxxxx
STAGE=dev
```

3、执行sls deploy联调部署后，访问url:https://service-xxxxxx-123456789.gz.apigw.tencentcs.com/release/feature，效果如下：
<img src="https://github.com/June1991/serverless-express/blob/master/doc/img/1593763296288.png" alt="1593763296288" style="zoom:40%;" />
至此联调完成，整个功能已经开发完毕。

#### 测试

1、把联调通过的dev分支合并到testing代码，进入测试。
<img src="https://github.com/June1991/serverless-express/blob/master/doc/img/1593686983115.png" alt="1593686983115" style="zoom:40%;" />
2、测试环境中的.env配置如下

```
TENCENT_SECRET_ID=xxxxxxxxxx
TENCENT_SECRET_KEY=xxxxxxxx
STAGE=testing
```

3、执行sls  deploy部署成功后，测试人员开始进行相关测试，直至功能稳定通过。


## 3.发布上线

测试通过后，我们把测试代码合并到master分支，现在准备发布上线。

<img src="https://github.com/June1991/serverless-express/blob/master/doc/img/1593687749028.png" alt="1593687749028" style="zoom:40%;" />



为了保证线上业务稳定，我们一般采取灰度发布的方式。

每次上线一个功能，执行`sls deploy`会部署到$latest版本上。我们将切部分流量在$latest版本上进行观察，然后逐步将流量切到$latest版本。当流量切到100%时，我们会固化这个版本，并将流量全部切到固化后的版本。

![1596441449319](https://github.com/June1991/serverless-express/blob/master/doc/img/1596441449319.png)

Serverless应用灰度发布采用云函数$default（默认流量）别名进行操作。步骤如下：

1、设置生产环境中的.env为：

```
TENCENT_SECRET_ID=xxxxxxxxxx
TENCENT_SECRET_KEY=xxxxxxxx
STAGE=prod
```

2、部署到线上环境$latest，并切换10%的流量在$latest版本：

```
sls deploy --inputs.traffic=0.1 #部署并切换10%流量到$latest版本上
```

3、对$latest版本进行一些监控与观察，等版本稳定之后把流量切到该版本上：

```
sls deploy --inputs.traffic=1.0 #部署并切换100%流量到$latest版本上
```

3、流量全部切换成功后，对于一个稳定版本，我们要把他做一个标记，以免后续发布新功能时，如果遇到线上问题，方便快速回退版本：

```
sls deploy --inputs.publish --inputs.traffic=0 #部署并发布函数版本v1，切换所有流量到版本v1
```

至此，我们完成了一个severless-express项目的开发和上线发布。



>?
>
>- 项目在部署过程中，根据不同环境设置不同的stage值，并让stage作为函数名称一部分，来实现环境隔离。
>- 灰度发布可以执行自定义脚本完成。



## 4.进阶文档

- [基于github 的自动化部署](https://github.com/June1991/serverless-express/blob/master/doc/%E5%9F%BA%E4%BA%8Egithub%E7%9A%84%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2.md)
- [基于coding 的自动化部署](https://github.com/June1991/serverless-express/blob/master/doc/%E5%9F%BA%E4%BA%8Ecoding%E7%9A%84%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2.md)

