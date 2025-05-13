#terraform #ec2 

```hcl
# Network Load Balancer (NLB) - TCP
resource "aws_lb" "web-lb" {
  name               = "web-lb"
  internal           = false
  load_balancer_type = "network"  # NLB
  subnets            = [aws_subnet.webPublic1.id, aws_subnet.webPublic2.id]
  security_groups    = [aws_security_group.sg-lb.id]
  enable_deletion_protection = false
}

resource "aws_lb_target_group" "web-tg" {
  name     = "web-tg"
  port     = 80
  protocol = "TCP"  # Must be TCP/UDP for NLB
  vpc_id   = aws_vpc.main.id
}

resource "aws_lb_listener" "web-lis" {
  load_balancer_arn = aws_lb.web-lb.arn
  port              = 80
  protocol          = "TCP"  # Must be TCP/UDP for NLB

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web-tg.arn
  }
}
```