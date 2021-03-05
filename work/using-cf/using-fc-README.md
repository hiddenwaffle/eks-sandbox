# using-fc

## Intro

Same as using-aws-cli, but I try to use CF for EKS too. But right now it is a simple S3 bucket example.

Example bucket commands:

```
aws cloudformation create-stack --stack-name eks-using-cf --template-body file://all.yaml

aws cloudformation describe-stacks

aws cloudformation update-stack --stack-name eks-using-cf --template-body file://all.yaml

aws s3 rm s3://<bucket-name> --recursive
aws cloudformation delete-stack --stack-name eks-using-cf
```
