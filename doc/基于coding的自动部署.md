#  Serverless-Coding CI/CD自动化部署 

在Serverless应用开发中，我们需要手动执行部署命令将开发项目部署到云端。本文将为大家讲解如何利用Coding的CI能力进行Serverless应用的自动化部署。

## 前提

- 已开通Coding账号。腾讯云用户可以通过 [CODING DevOps](https://console.cloud.tencent.com/coding) 快速开通。
- 已创建Serverless应用项目。如果您未创建Serverless 应用项目，请参考[示例链接 >>](https://github.com/June1991/serverless-express)创建您的serverless项目并创建各个环境与分支。
- 已托管您的Serverless项目到Coding/Github/Gitlab/码云。

## 操作场景

在开发测试阶段，为了方便开发、测试和调试，希望代码每次提交后进行自动化部署。操作如下：

1. 创建您的Coding Devops项目。

   ![1596444814258]( https://github.com/June1991/serverless-express/blob/master/doc/img/1596444814258.png )

2. 创建一个构建计划，选择自定义构建过程。

   ![1596446336910](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596446336910.png)

3. 配置构建计划。

   1. 基础信息配置。本例中配置github仓库：June1991/express-demo。

      ![1596445194613](https://github.com/June1991/serverless-express/blob/master/doc/img/1596445194613.png)

   2. 触发规则配置。本例中配置代码推送到dev分支时触发构建。

      ![1596445230077](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1596445230077.png)

   3. 环境变量配置。本例中配置STAGE变量为部署环境dev，TENCENT_CLOUD_API_CRED为腾讯云账号密钥（密钥配置路径：左下角项目设置-》开发者选项-》凭据管理-》录入凭据-》腾讯云API密钥）。

      ![1596445800067](https://github.com/June1991/serverless-express/blob/master/doc/img/1596445800067.png)

   4. 流程配置。

      ```
      pipeline {
        agent any
        stages {
          stage('检出') {
            steps {
              checkout([$class: 'GitSCM', branches: [[name: env.GIT_BUILD_REF]],
                  userRemoteConfigs: [[url: env.GIT_REPO_URL, credentialsId: env.CREDENTIALS_ID]]])
            }
          }
          stage('安装依赖') {
            steps {
              echo '安装依赖中...'
              sh 'npm i -g serverless'
              sh 'npm install'
              echo '安装依赖完成.'
            }
          }
          stage('部署') {
            steps {
              echo '部署中...'
      
              withCredentials([
                cloudApi(
                  credentialsId: "${env.TENCENT_CLOUD_API_CRED}",
                  secretIdVariable: 'TENCENT_SECRET_ID',
                  secretKeyVariable: 'TENCENT_SECRET_KEY'
                ),
              ]) {
      
                   // 生成凭据文件
                   sh 'echo "TENCENT_SECRET_ID=${TENCENT_SECRET_ID}\nTENCENT_SECRET_KEY=${TENCENT_SECRET_KEY}" > .env'
                   // 部署
                   sh 'sls deploy --debug'   
                   // 移除凭据
                   sh 'rm .env 
              }
      
              echo '部署完成'
            }
          }
        }
      }
      ```

      自此，开发者每次提交代码到dev分支时，就会自动部署。