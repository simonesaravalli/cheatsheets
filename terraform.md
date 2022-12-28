## Import existing AWS RDS Postgres instance in Terraform under module jms-olap

```
terraform init -var-file=input-eu-west-1-test.tfvars
terraform plan -var-file=input-eu-west-1-test.tfvars
terraform import -var-file=input-eu-west-1-test.tfvars module.jms-olap.module.db_instance.aws_db_instance.this rds-eu-west-1-test-jms-olap
```