#terraform #ec2

```hcl
resource "aws_launch_template" "app-template" {
  name_prefix            = "app"
  image_id               = var.linux-ami
  instance_type          = var.instance-type
  vpc_security_group_ids = [aws_security_group.sg-app.id]
}

resource "aws_autoscaling_group" "autoscale-web" {
  name                 = "ag-1"
  desired_capacity     = 2
  max_size             = 4
  min_size             = 1
  health_check_type    = "EC2"
  termination_policies = ["OldestInstance"]
  vpc_zone_identifier  = [aws_subnet.webPublic1.id, aws_subnet.webPublic2.id]

  launch_template {
    id      = aws_launch_template.web-template.id
    version = aws_launch_template.web-template.latest_version
  }
}
```