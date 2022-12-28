## TAag an untagged image in AWS ECR

Tag an untagged ECR image:

```
MANIFEST=$(aws ecr batch-get-image --repository-name sandbox --image-ids imageDigest=sha256:e226e9aaa12beb32bfe65c571cb60605b2de13338866bc832bba0e39f6819365 --query 'images[].imageManifest' --output text)
aws ecr put-image --repository-name sandbox --image-tag backup --image-manifest "$MANIFEST"
```

Then download it:

```
docker pull myarn.amazonaws.com/sandbox:backup
```