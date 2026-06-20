# Threat Composer on ECS Fargate

Deployment of the open-source [Threat Composer](https://github.com/awslabs/threat-composer)
application (a threat modeling tool by AWS Labs) on ECS Fargate, using Terraform
for infrastructure as code and GitHub Actions for CI/CD.

## Goal

A learning pet project to practice:

- Terraform (modular structure, remote state)
- AWS networking (VPC, ALB, ECS Fargate)
- CI/CD (GitHub Actions)
- DevSecOps (image and infrastructure scanning)

## Architecture

```
                          Internet
                             |
                             v
                    +-----------------+
                    |   ALB (public)  |
                    +--------+--------+
                             |
                +------------+------------+
                |                         |
        +-------v-------+         +-------v-------+
        | Fargate Task  |         | Fargate Task  |
        | (private,     |         | (private,     |
        |  AZ-a)        |         |  AZ-b)        |
        +---------------+         +---------------+
```

- ALB sits in public subnets and receives traffic from the internet
- ECS Fargate tasks run in private subnets — no direct internet access
- Multi-AZ for high availability
- (Later) HTTPS via ACM + Route53, once a domain is purchased

## Stack

| Category          | Tool                                     |
| ----------------- | ---------------------------------------- |
| Containerization  | Docker (multi-stage build, non-root)     |
| IaC               | Terraform                                |
| Compute           | AWS ECS Fargate                          |
| Networking        | VPC, ALB, Security Groups                |
| CI/CD             | GitHub Actions                           |
| Security scanning | Trivy (image), Checkov/tfsec (Terraform) |

## Repository structure

```
.
├── .github/workflows/   # CI/CD pipelines
├── app/                 # Dockerfile, nginx config
├── terraform/
│   ├── modules/
│   │   ├── vpc/
│   │   ├── security-groups/
│   │   ├── alb/
│   │   └── ecs/
│   ├── main.tf
│   ├── variables.tf
│   └── provider.tf
└── README.md
```

## Status

Work in progress — learning project, built step by step.

- [ ] Dockerfile for the application
- [ ] Terraform: VPC module
- [ ] Terraform: Security Groups
- [ ] Terraform: ALB
- [ ] Terraform: ECS Fargate + ECR
- [ ] Terraform: remote backend (S3)
- [ ] GitHub Actions: build & push image
- [ ] GitHub Actions: terraform plan/apply
- [ ] Security scanning (Trivy, Checkov)
- [ ] HTTPS (Route53 + ACM) — after buying a domain
- [ ] CloudWatch monitoring

## Cost notes

ALB and NAT Gateway are not covered by the free tier and cost money even when
idle (~$16/mo ALB, ~$32/mo NAT Gateway). Infrastructure should only be kept up
while actively working on it and torn down (`terraform destroy`) afterward.
