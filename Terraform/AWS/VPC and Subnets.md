#terraform #ec2

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

# Public subnets
resource "aws_subnet" "webPublic1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "eu-west-1a"  # Explicit AZ assignment
}
```
