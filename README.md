# Web Front-End Module

This module creates a scalable web front-end infrastructure with the following components:

## Resources Created

- **Security Groups**: NLB security group and EC2 security group
- **Launch Template**: EC2 launch template with custom user data
- **Auto Scaling Group**: ASG for managing EC2 instances
- **Network Load Balancer**: Internet-facing NLB for traffic distribution
- **Target Group**: Load balancer target group
- **Autoscaling Attachment**: Links ASG to target group

## Usage

```terraform
module "web_front_end" {
  source = "./modules/web-front-end"

  app_port                = 80
  autoscale_group_size    = 2
  autoscale_group_min_max = {
    min = 1
    max = 3
  }
  environment         = "dev"
  instance_type       = "t3.micro"
  launch_template_ami = "ami-xxxxxxxxx"
  prefix              = "tw"
  public_subnet_ids   = ["subnet-xxxxx", "subnet-yyyyy"]
  vpc_id              = "vpc-xxxxx"
  user_data_contents  = base64encode("#!/bin/bash\necho 'Hello World'")
}
```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| app_port | Port the application listens on | number | 80 | no |
| autoscale_group_min_max | Min/max size for autoscale group | object | n/a | yes |
| autoscale_group_size | Default size of autoscale group | number | n/a | yes |
| environment | Environment of all resources | string | n/a | yes |
| instance_type | Instance type for Autoscale group | string | "t3.micro" | no |
| launch_template_ami | AMI ID to use for the launch template | string | n/a | yes |
| prefix | Prefix to use for all resources | string | n/a | yes |
| public_subnet_ids | List of public subnet IDs | list(string) | n/a | yes |
| user_data_contents | User data script contents (base64encoded) | string | n/a | yes |
| vpc_id | VPC ID where resources will be deployed | string | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| autoscaling_group_name | The name of the autoscaling group |
| lb_public_dns | The public DNS name of the load balancer |

## Features

- **Security Isolation**: EC2 instances only accept traffic from NLB
- **Health Checks**: ELB health checks ensure only healthy instances receive traffic
- **Auto Scaling**: Supports dynamic scaling based on policies (defined outside module)
- **Multi-AZ**: Distributes instances across multiple availability zones
- **Customizable**: Flexible configuration through input variables
