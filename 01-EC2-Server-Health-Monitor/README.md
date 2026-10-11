# AWS EC2 CPU Health Monitor

## Overview

This project demonstrates infrastructure monitoring and alerting using an Ubuntu EC2 instance, Amazon CloudWatch, and Amazon SNS.

The objective is to monitor EC2 CPU utilization, configure a CloudWatch alarm, and receive email notifications when CPU utilization crosses a defined threshold.

The lab also explores Linux CPU statistics and troubleshooting discrepancies between operating-system metrics and AWS CloudWatch metrics.

## Objectives

- Launch and access an Ubuntu EC2 instance.
- Explore EC2 CPU utilization metrics in CloudWatch.
- Configure a CloudWatch alarm.
- Integrate the alarm with an SNS topic.
- Test the alarm by generating CPU load.
- Investigate CPU utilization, steal time, and load average on Linux.

## Architecture

```text
Ubuntu EC2 Instance
        |
        | EC2 CPUUtilization metric
        v
Amazon CloudWatch
        |
        v
CloudWatch Alarm
        |
        v
Amazon SNS Topic
        |
        v
Email Notification
```

## AWS Services and Technologies

| Component         | Purpose                                             |
| ----------------- | --------------------------------------------------- |
| Amazon EC2        | Hosts the Ubuntu Linux instance                     |
| Amazon CloudWatch | Monitors EC2 metrics and evaluates alarm conditions |
| Amazon SNS        | Sends email notifications for alarm state changes   |
| Linux CLI         | Generates test load and supports troubleshooting    |

## Monitoring and Alerting

A CloudWatch alarm named `aws-health-monitor-high-cpu` was configured to monitor the EC2 `CPUUtilization` metric.

The threshold was lowered to 5% during testing to demonstrate the alarm workflow in the lab environment.

The alarm entered the `ALARM` state. Email delivery through SNS should be recorded as verified only after the notification is received.

## Troubleshooting Experience

EC2 Instance Connect initially failed, so an SSH client was used to access the instance.

During CPU load testing, Linux `top` showed high CPU steal time while CloudWatch CPU utilization remained around 10%. This highlighted the importance of investigating operating-system CPU statistics alongside cloud monitoring metrics.

See [Troubleshooting](docs/troubleshooting.md) for details.

## Cleanup

Stop all CPU load processes after testing. When the lab is complete, terminate the EC2 instance and remove any lab resources that are no longer required.

## Future Improvements

- Add custom CloudWatch metrics for memory and disk utilization.
- Automate health checks and report generation.
- Explore infrastructure provisioning with Terraform.
- Extend the lab with additional AWS services.

## Disclaimer

This is a hands-on learning lab, not a production-ready monitoring platform.
