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

<!-- BEGIN_TF_DOCS -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_app_port"></a> [app\_port](#input\_app\_port) | Port the application listens on | `number` | `80` | no |
| <a name="input_autoscale_group_min_max"></a> [autoscale\_group\_min\_max](#input\_autoscale\_group\_min\_max) | The minimum and maximum size for the autoscale group. | <pre>object({<br/>    min = number<br/>    max = number<br/>  })</pre> | n/a | yes |
| <a name="input_autoscale_group_size"></a> [autoscale\_group\_size](#input\_autoscale\_group\_size) | Default size of autoscale group. | `number` | n/a | yes |
| <a name="input_environment"></a> [environment](#input\_environment) | (Required) Environment of all resources | `string` | n/a | yes |
| <a name="input_instance_tags"></a> [instance\_tags](#input\_instance\_tags) | Additional tags to apply to EC2 instances launched by the Auto Scaling Group | `map(string)` | `{}` | no |
| <a name="input_instance_type"></a> [instance\_type](#input\_instance\_type) | Instance type for Autoscale group | `string` | `"t3.micro"` | no |
| <a name="input_launch_template_ami"></a> [launch\_template\_ami](#input\_launch\_template\_ami) | AMI ID to use for the launch template | `string` | n/a | yes |
| <a name="input_prefix"></a> [prefix](#input\_prefix) | (Required) Prefix to use for all resources in this module. | `string` | n/a | yes |
| <a name="input_public_subnet_ids"></a> [public\_subnet\_ids](#input\_public\_subnet\_ids) | List of public subnet IDs for the autoscale group and NLB. | `list(string)` | n/a | yes |
| <a name="input_test4"></a> [test4](#input\_test4) | Test variable | `string` | `null` | no |
| <a name="input_user_data_contents"></a> [user\_data\_contents](#input\_user\_data\_contents) | User data script contents for the launch template. | `string` | n/a | yes |
| <a name="input_vpc_id"></a> [vpc\_id](#input\_vpc\_id) | VPC ID where resources will be deployed. | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_autoscaling_group_name"></a> [autoscaling\_group\_name](#output\_autoscaling\_group\_name) | The name of the autoscaling group |
| <a name="output_lb_public_dns"></a> [lb\_public\_dns](#output\_lb\_public\_dns) | The public DNS name of the load balancer |
<!-- END_TF_DOCS -->

## Features

- **Security Isolation**: EC2 instances only accept traffic from NLB
- **Health Checks**: ELB health checks ensure only healthy instances receive traffic
- **Auto Scaling**: Supports dynamic scaling based on policies (defined outside module)
- **Multi-AZ**: Distributes instances across multiple availability zones
- **Customizable**: Flexible configuration through input variables
