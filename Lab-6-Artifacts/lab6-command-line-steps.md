# Lab 6 - Automated Deployments - Command Line Steps

## 6.1	Create an AWS CodePipeline pipeline

aws codecommit create-repository

aws codebuild create-project

aws codepipeline create-pipeline --cli-input-json file://MySecondPipeline.json
