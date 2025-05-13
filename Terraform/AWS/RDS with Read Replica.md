#terraform #rds

```hcl
resource "aws_db_instance" "prodb1" {
  identifier = "prodb1"
  allocated_storage      = 10
  db_name                = "prod"
  engine                 = "mysql"
  engine_version         = "5.7"
  instance_class         = var.db-instance-type
  username               = "admin"
  password               = var.db-password
  parameter_group_name   = "default.mysql5.7"
  multi_az               = true
  db_subnet_group_name   = aws_db_subnet_group.main-private1.id
  vpc_security_group_ids = [aws_security_group.sg-db.id]
  skip_final_snapshot    = true
  backup_retention_period = 7
}

resource "aws_db_instance" "prod1-replica" {
  identifier             = "prod1-replica"
  replicate_source_db    = aws_db_instance.prodb1.arn
  instance_class         = var.db-instance-type
  db_subnet_group_name   = aws_db_subnet_group.main-private2.name
  vpc_security_group_ids = [aws_security_group.sg-db.id]
  skip_final_snapshot    = true
  apply_immediately      = true
  availability_zone      = "eu-west-1b"
}

resource "aws_db_subnet_group" "main-private1" {
  name       = "main-private1"
  subnet_ids = [aws_subnet.dbPrivate1.id, aws_subnet.dbPrivate2.id]
}

resource "aws_db_subnet_group" "main-private2" {
  name       = "main-private2"
  subnet_ids = [aws_subnet.dbPrivate2.id, aws_subnet.dbPrivate1.id]
}
```