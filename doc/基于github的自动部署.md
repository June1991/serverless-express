#  Serverless-Github Action自动化部署 



## 前提

- 已创建Serverless应用项目。如果您未创建Serverless 应用项目，请参考[示例链接 >>](https://github.com/June1991/serverless-express)创建您的serverless项目并创建各个环境与分支。
- 已托管您的Serverless项目到Github。

## 操作场景

本文将为大家讲解如何利用Github的CI能力进行Serverless应用的自动化部署。

```
name: deploy serverless


on:
  push:
    branches:
      - master 
  
jobs:
  test:
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
        env:
          STAGE: prod
          SERVERLESS_PLATFORM_VENDOR: tencent
          TENCENT_SECRET_ID: ${{ secrets.TENCENT_SECRET_ID }}
          TENCENT_SECRET_KEY: ${{ secrets.TENCENT_SECRET_KEY }}
        
```

