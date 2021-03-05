# eks-sandbox

## Reference

## Set up AWS CLI

Run the custom CLI container

```
docker build -t my-aws-cli my-aws-cli
docker run -it --rm -v ${PWD}/work:/work -w /work --entrypoint /bin/bash my-aws-cli
```

Access keys

See the "Access keys" section in: https://console.aws.amazon.com/iam/home?region=us-west-2#/security_credentials

Region name
```
us-west-2
```

Login with the AWS CLI

```
aws configure
```

Look around

```
aws help
aws eks help
```

From here, continue on to `work/using-aws-cli` or `work/using-cf` or other trail to set up an EKS cluster
