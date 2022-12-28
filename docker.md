## Build Docker image and push it to local Docker Registry

```
docker build -t <local docker registry hostname>:5000/<docker image name>:<docker image tag> -f Dockerfile-local .
docker push <local docker registry hostname>:5000/<docker image name>:<docker image tag>
```

## Build Docker image, push it to local Docker Registry and also to AWS ECR

```
docker build -t <local docker registry hostname>:5000/<docker image name>:<docker image tag>-aws -f Dockerfile-aws .
docker push <local docker registry hostname>:5000/<docker image name>:<docker image tag>-aws

aws ecr get-login-password --region <aws region> | docker login --username AWS --password-stdin <aws account ID>.dkr.ecr.<aws region>.amazonaws.com
(use AWS IAM credentials of a user that can access to AWS ECR)

docker tag <local docker registry hostname>:5000/<docker image name>:<docker image tag>-aws <aws account ID>.dkr.ecr.<aws region>.amazonaws.com/ecr-<aws region>-<docker image name>:latest

docker push <aws account ID>.dkr.ecr.<aws region>.amazonaws.com/ecr-<aws region>-<docker image name>:latest
```