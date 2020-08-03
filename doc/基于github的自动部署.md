#  Serverless-Github Action自动化部署 

在Serverless应用开发中，我们需要手动执行部署命令将开发项目部署到云端。本文将为大家讲解如何利用Github的CI能力进行Serverless应用的自动化部署。

## 前提

- 已创建Serverless应用项目。如果您未创建Serverless 应用项目，请参考[示例链接 >>](https://github.com/June1991/serverless-express)创建您的serverless项目并创建各个环境与分支。
- 已托管您的Serverless项目到Github。

## 操作步骤

在开发测试阶段，为了方便开发、测试和调试，希望代码每次提交后进行自动化部署。操作如下：

1. 选取一个你需要执行自动化部署的分支。例如我选择dev分支。

2. 在该分支下创建您的action。（参考github官网[配置工作流程](https://docs.github.com/cn/actions/configuring-and-managing-workflows/configuring-a-workflow)）  

   ![1596438834138](https://github.com/June1991/serverless-express/blob/master/doc/img/1596438834138.png)

   **注意：github规定如果事件发生在特定仓库分支上，则工作流程文件必须存在于该分支的仓库中 。**

3. 配置腾讯云密钥。参考github官网[使用变量和密码](https://docs.github.com/cn/actions/configuring-and-managing-workflows/using-variables-and-secrets-in-a-workflow)

   ![1596439648697](https://github.com/June1991/serverless-express/blob/master/doc/img/1596439648697.png)

4. 配置action部署步骤。

```
# 当代码推动到dev分支时，执行当前工作流程
# 更多配置信息: https://docs.github.com/cn/actions/getting-started-with-github-actions

name: deploy serverless

on: #监听的事件和分支配置
  push:
    branches:
      - dev 
  
jobs:
  test: #配置单元测试
    name: test
    runs-on: ubuntu-latest
    steps:
      - name: unit test
        run: '' 
  deploy:
    name: deploy serverless
    runs-on: ubuntu-latest
    needs: [test]
    steps:
      - name: clone local repository
        uses: actions/checkout@v2
      - name: install serverless
        run: npm install -g serverless
      - name: install dependency
        run: npm install
      - name: build
        run: npm build
      - name: deploy serverless
        run: sls deploy --debug
        env: # 环境变量
          STAGE: dev #您的部署环境
          SERVERLESS_PLATFORM_VENDOR: tencent #serverless海外默认为aws，配置为腾讯
          TENCENT_SECRET_ID: ${{ secrets.TENCENT_SECRET_ID }} #您的腾讯云账号sercret ID
          TENCENT_SECRET_KEY: ${{ secrets.TENCENT_SECRET_KEY }}#您的腾讯云账号sercret key
          
```

自此，开发者每次提交代码到dev分支时，就会自动部署。