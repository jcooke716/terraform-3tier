# terraform-3tier

~ This is a repository for my terraform projects. 

~ I will use terraform to create resources in AWS

## Project overview

An AWS infrastructure-as-code lab that separates public entry points, private application servers, and private database networking across two Availability Zones. It explores how traffic routing, security groups, load balancing, and scaling fit together in a three-tier environment.

## Architecture defined in code

| Layer | Resources and behavior |
| --- | --- |
| Network | A VPC, two public subnets, two private application subnets, and two private data subnets in `us-east-1a` and `us-east-1b`. |
| Public entry | An internet-facing Application Load Balancer with HTTP-to-HTTPS redirection, an HTTPS listener, and a target group. |
| Application | A launch template and Auto Scaling group in the private application subnets; desired capacity 2, minimum 1, maximum 4. |
| Private egress | One NAT gateway per Availability Zone, with private route tables and subnet associations. |
| Database | An RDS instance restored from an existing manual snapshot and placed in a database subnet group. |
| DNS and notifications | A Route 53 alias record, an SNS email subscription, and instance lifecycle notifications. |

The application security group accepts web traffic from the load balancer security group. The database security group accepts port 3306 from the application security group.

## File map

| File | Responsibility |
| --- | --- |
| [vpc.tf](vpc.tf) | VPC, subnets, internet gateway, and public routing. |
| [nat-gateway.tf](nat-gateway.tf) | NAT gateways, Elastic IPs, and private routing. |
| [security-group.tf](security-group.tf) | Load balancer, application, database, and SSH security groups. |
| [alb.tf](alb.tf) | Load balancer, listeners, target group, and health check settings. |
| [asg.tf](asg.tf) | Launch template, Auto Scaling group, attachment, and notifications. |
| [rds.tf](rds.tf) | Snapshot lookup and database instance. |
| [route-53.tf](route-53.tf), [sns.tf](sns.tf) | DNS alias and notification resources. |
| [main.tf](main.tf), [variable.tf](variable.tf), [output.tf](output.tf) | Provider, S3 state backend, inputs, and outputs. |

## Environment requirements

Review the account-specific defaults before planning a deployment. The configuration expects an AWS profile named `terraform-user`, an existing S3 state bucket, a suitable application AMI, an EC2 key pair, an RDS snapshot, an ACM certificate, and a Route 53 hosted zone.

The committed provider lock selects AWS provider 3.74.0. Review compatibility before upgrading because the configuration includes older provider arguments.

For a configuration review without initializing the configured remote state backend:

```bash
terraform init -backend=false
terraform validate
terraform fmt -check
```

A deployment plan requires adapting the backend and resource identifiers to the target AWS account. No plan or deployment success is asserted by this README.

## Design boundaries

- The launch template uses an existing AMI; this repository does not build the application image or install an application through user data.
- RDS Multi-AZ is configurable and defaults to `false`; distributing subnets across zones does not make the default database highly available.
- The SSH security group is defined, but a bastion EC2 instance is not.
- The VPC and public-subnet outputs currently contain quoted reference strings instead of evaluated resource IDs.
- Review the broad default SSH source range and the database deletion settings before provisioning resources.

## Portfolio focus

AWS network segmentation, resource dependencies, load balancer and Auto Scaling integration, snapshot-based database provisioning, DNS, and notification design. This project covers the broader application infrastructure, complementing the narrower network foundation and single-instance labs.
