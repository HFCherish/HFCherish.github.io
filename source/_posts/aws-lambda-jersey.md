---
title: 在 aws lambda 中应用 jersey
toc: true
tags:
  - aws
  - aws lambda
date: 2018-06-27 10:39:55
---


使用 [aws serverless java container](https://github.com/awslabs/aws-serverless-java-container) 实现。

# 使用 [aws cli](https://aws.amazon.com/cn/cli/)

## 创建项目，配置 aws cli
```sh
# 利用原型创建项目
$ mvn archetype:generate -DgroupId=my.service -DartifactId=my-service -Dversion=1.0-SNAPSHOT \
       -DarchetypeGroupId=com.amazonaws.serverless.archetypes \
       -DarchetypeArtifactId=aws-serverless-jersey-archetype \
       -DarchetypeVersion=1.1.3


# 安装 aws cli
$ pip install awscli
# 配置 credentials
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json


# 写代码，测试……
```

aws 安装之后一般要配置 credentials，详情可参见 [aws cli 配置](https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-chap-getting-started.html)

## 打包部署

```sh
# package
$ mvn clean package


# 上传 package 到 s3（需要先创建 s3 bucket）
$ aws s3 mb s3://BUCKET_NAME
$ aws cloudformation package --template-file sam.yaml --output-template-file output-sam.yaml --s3-bucket <YOUR S3 BUCKET NAME>


# 部署到 aws lambda
$ aws cloudformation deploy --template-file output-sam.yaml --stack-name your-stack-name --capabilities CAPABILITY_IAM
```

## 查看部署结果

```sh
# 可查看 stack 详情
$ aws cloudformation describe-stacks --stack-name your-stack-name
{
    "Stacks": [
        {
            "StackId": "arn:aws:cloudformation:us-west-2:xxxxxxxx:stack/ServerlessJerseyApi/xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx", 
            "Description": "AWS Serverless Jersey API - learning.aws::aws-lambda-jersey", 
            "Tags": [], 
            "Outputs": [
                {
                    "Description": "URL for application",
                    "ExportName": "AwsLambdaJerseyApi",  
                    "OutputKey": "AwsLambdaJerseyApi",
                    "OutputValue": "https://xxxxxxx.execute-api.us-west-2.amazonaws.com/Prod/ping"
                }
            ], 
            "CreationTime": "2016-12-13T22:59:31.552Z", 
            "Capabilities": [
                "CAPABILITY_IAM"
            ], 
            "StackName": "ServerlessJerseyApi", 
            "NotificationARNs": [], 
            "StackStatus": "UPDATE_COMPLETE"
        }
    ]
}

# 根据生成的链接访问 api
$ curl https://xxxxxxx.execute-api.us-west-2.amazonaws.com/Prod/ping
```

# 使用 [sam-cli](https://github.com/awslabs/aws-sam-cli)

aws-cli 只能部署到远端去查看运行效果。而 sam-cli 则可以本地 simulate 一个 lambda 环境，从而实现本地调试。

## 创建项目

aws-sam-cli 有个 `sam init --runtime java`，可以创建一个 sample 项目，但是目前支持的 template 并不包含 jersey 的。所以要创建项目还是通过 mvn archetype。

（所有支持的 runtime 可以查看 [project status](https://github.com/awslabs/aws-sam-cli#project-status)）

但是 `sam init` 创建了一个 `template.yaml`，这是之后所有 sam 命令的依据。这个文件会被翻译为 `mvn archetype` 创建的 `sam.yaml`。因为其后台是利用 `aws cli` 运行的。

所以我们需要在 `mvn archetype` 创建了原型之后，手动依据 `sam.yaml` 创建 `template.yaml`

```sh
# 利用原型创建项目，配置 aws-cli，同上

# 创建 template.yaml
$ cp sam.yaml template.yaml
```

## 部署项目

```sh
# package
$ mvn clean package


# 本地测试运行
$ sam local start-api


# 上传 package 到 s3（需要先创建 s3 bucket）
$ aws s3 mb s3://BUCKET_NAME
$ sam package \
    --template-file template.yaml \
    --output-template-file packaged.yaml \
    --s3-bucket YOUR_S3_BUCKET_NAME

# 部署到 aws lambda
$ sam deploy \
    --template-file packaged.yaml \
    --stack-name your-stack-name \
    --capabilities CAPABILITY_IAM \
    --parameter-overrides MyParameterSample=MySampleValue
```

如果在本地运行时，报错 `No class found....`，一般是因为 docker 的原因，我最后是通过重装 docker 解决的。由于 aws-sam-cli 使用 docker 去起一个 lambda 容器环境，所以可能是由于 credentials 等获取不到的原因？，总之可能会挂掉。可参见 [stackoverflow ticket](https://github.com/docker/for-mac/issues/488)，

## 查看部署结果

这里跟上边是一样的，我们可以只看一部分

```sh
# 可查看 stack 详情
$ aws cloudformation describe-stacks \
	--stack-name your-stack-name \
	--query 'Stacks[].Outputs'
[
	[
	    {
	        "Description": "URL for application",
	        "ExportName": "AwsLambdaJerseyApi",  
	        "OutputKey": "AwsLambdaJerseyApi",
	        "OutputValue": "https://xxxxxxx.execute-api.us-west-2.amazonaws.com/Prod/ping"
	    }
	]
]

# 根据生成的链接访问 api
$ curl https://xxxxxxx.execute-api.us-west-2.amazonaws.com/Prod/ping¡
```