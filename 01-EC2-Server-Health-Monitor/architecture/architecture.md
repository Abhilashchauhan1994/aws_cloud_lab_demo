# Architecture — AWS EC2 CPU Server Health Monitor

## 1. Overview

This project demonstrates a basic AWS CPU monitoring and alerting solution using Amazon EC2, Amazon CloudWatch, and Amazon Simple Notification Service (SNS).

An Ubuntu EC2 instance generates the infrastructure metrics monitored by CloudWatch. A CloudWatch alarm evaluates the instance's CPU utilization against a configured threshold. When the alarm enters the `ALARM` state, SNS can deliver a notification to a subscribed email address.

The purpose of this lab is to understand AWS infrastructure monitoring, alert configuration, notification delivery, and basic Linux CPU troubleshooting.

## 2. Architecture Diagram

```text
              Ubuntu EC2 Instance
                       |
                       | EC2 CPUUtilization metric
                       v
                Amazon CloudWatch
                       |
                       v
             CloudWatch CPU Alarm
          aws-health-monitor-high-cpu
                       |
                       | Alarm state change
                       v
                 Amazon SNS
            aws-health-monitor-alerts
                       |
                       v
                Email Notification
```

**Note:** This diagram represents the monitoring and alerting flow. The Linux CPU load was generated manually during testing. No custom CloudWatch metrics or Python-based monitoring agent were implemented in this lab.

## 3. AWS Components

### Amazon EC2

Amazon EC2 provides the Ubuntu Linux virtual machine used for this exercise.

The instance was used to:

- Practise connecting to a Linux server through SSH.
- Observe CPU statistics using Linux commands.
- Generate CPU load for alarm testing.
- Provide the EC2 instance metrics monitored by CloudWatch.

### Amazon CloudWatch

CloudWatch collects standard EC2 metrics, including `CPUUtilization`.

The metric was located under the EC2 per-instance metrics in the CloudWatch console. It was used as the input for the CPU alarm.

### CloudWatch Alarm

The alarm evaluates the configured CPU utilization threshold and changes state when its evaluation criteria are met.

Alarm name:

`aws-health-monitor-high-cpu`

During testing, the threshold was lowered to 5% to demonstrate the alarm workflow. This is a lab-specific threshold, not a recommended production setting.

### Amazon SNS

Amazon SNS provides the notification mechanism connected to the CloudWatch alarm.

SNS topic:

`aws-health-monitor-alerts`

An email endpoint was configured for notifications. The subscription must be confirmed before SNS can deliver email notifications to that endpoint.

## 4. Monitoring Flow

1. The Ubuntu EC2 instance runs in AWS.
2. EC2 publishes its standard CPU utilization metric to CloudWatch.
3. CloudWatch evaluates the metric against the alarm configuration.
4. When the configured threshold is breached, the alarm transitions to `ALARM`.
5. The alarm notification action publishes a message to the configured SNS topic.
6. SNS sends an email notification to the confirmed email subscription.

## 5. Design Decisions

### Why CloudWatch?

CloudWatch provides native monitoring for EC2 without requiring us to deploy and maintain a separate monitoring server.

### Why SNS?

SNS separates alarm evaluation from notification delivery and supports email notifications for operational alerts.

### Why generate CPU load?

A controlled workload provides a practical way to test whether the monitoring and notification configuration responds to a threshold breach.

### Why use Linux troubleshooting commands?

Commands such as `top`, `uptime`, and `ps` help investigate operating-system behaviour and compare it with cloud-level monitoring data.

## 6. Observations and Limitations

During the CPU load test, Linux `top` reported high CPU steal time while CloudWatch CPU utilization remained around 10%.

This demonstrated that Linux CPU statistics and CloudWatch metrics must be interpreted in context. High steal time indicates CPU time the virtual machine was unable to use because the hypervisor scheduled other work. It does not, by itself, prove the exact cause of the contention.

This lab monitors standard EC2 CPU utilization only. It does not currently publish custom memory, disk, or application-health metrics to CloudWatch.

## 7. Future Improvements

Potential extensions include:

- Publishing custom memory and disk utilization metrics.
- Adding alarms for additional operational conditions.
- Automating repeatable deployment and cleanup.
- Integrating the monitoring workflow with an incident management system.
- Exploring infrastructure as code using AWS CloudFormation or Terraform.

## 8. Related Documentation

- [Project Overview](../README.md)
- [CloudWatch Alarm Configuration](../cloudwatch/alarms.md)
- [Deployment Guide](../docs/deployment.md)
- [Troubleshooting Notes](../docs/troubleshooting.md)
