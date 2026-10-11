# AWS Cloud Hands-On Labs

A practical repository for learning AWS through hands-on labs focused on cloud infrastructure, monitoring, automation, networking, serverless services, and troubleshooting.

The goal is to build practical cloud engineering and Site Reliability Engineering (SRE) skills by implementing solutions, investigating failures, validating results, and documenting lessons learned.

## Learning Objectives

- Gain hands-on experience with core AWS services.
- Deploy and troubleshoot cloud infrastructure.
- Implement monitoring, alerting, and automation.
- Understand cloud networking and connectivity.
- Explore serverless applications and container orchestration.
- Practice incident investigation and operational troubleshooting.
- Build reproducible projects for a technical portfolio.

## Lab Roadmap

| # | Lab | Primary Services / Technologies | Status |
|---|---|---|---|
| 01 | [EC2 CPU Server Health Monitor](./01-EC2-CPU-Server-Health-Monitor/) | EC2, CloudWatch, SNS, Linux | Implemented; documentation in progress |
| 02 | S3 Backup and Recovery Automation | S3, AWS CLI | Planned |
| 03 | VPC Networking and Connectivity Troubleshooting | VPC, Subnets, Route Tables, Security Groups | Planned |
| 04 | Serverless Log Processing | Lambda, CloudWatch Logs | Planned |
| 05 | Deploy an Application on Amazon EKS | EKS, Kubernetes, container tooling | Planned |
| 06 | AWS Incident Troubleshooting Lab | EC2, CloudWatch, VPC | Planned |

*The roadmap may be adjusted based on AWS Sandbox permissions, available services, quotas, and cost limits.*

## How Each Lab Works

Each lab aims to follow this workflow:

1. **Problem:** Define a realistic cloud or operational challenge.
2. **Design:** Identify the architecture and required services.
3. **Implementation:** Build the solution using the AWS Console, CLI, or scripts.
4. **Validation:** Test whether the solution behaves as expected.
5. **Troubleshooting:** Investigate failures using evidence and logs.
6. **Documentation:** Record setup steps, findings, and lessons learned.
7. **Cleanup:** Remove unnecessary resources after completing the exercise.

Lab directories will be added as the projects are implemented.

## Sandbox Compatibility

Labs will be designed to work within the permissions and restrictions of the available AWS learning environment wherever possible.

- Avoid requiring custom IAM role or policy creation unless the sandbox explicitly supports it.
- Prefer preconfigured permissions and existing execution roles when available.
- Check service availability, quotas, and resource restrictions before implementation.
- Provide alternatives when a service or operation is restricted.
- Do not assume that every AWS service is enabled in every sandbox.

Some services, including Lambda and EKS, may still require preconfigured roles or permissions. Their implementation will depend on what the sandbox allows.

## Cost and Security

- Check pricing and usage limits before deploying resources.
- Use appropriately sized resources for experiments.
- Restrict network access to trusted sources.
- Never commit credentials, private keys, or secrets.
- Clean up resources after each lab.
- Use production-appropriate security settings when adapting demonstrations.

## Technologies

- Amazon Web Services
- Linux and SSH
- AWS Management Console and AWS CLI
- Cloud monitoring and troubleshooting tools
- Containers and Kubernetes, where supported

## Disclaimer

This repository is intended for educational and portfolio purposes. Lab designs may be simplified to fit sandbox limitations. Always validate permissions, security, reliability, and cost implications before using these configurations in production.
