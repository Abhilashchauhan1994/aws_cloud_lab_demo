# CloudWatch CPU Monitoring and Alarm Configuration

## 1. Objective

This document describes how Amazon CloudWatch was configured to monitor CPU utilization for the EC2 instance and trigger notifications through Amazon SNS when the configured threshold was breached.

## 2. EC2 Metric

The CloudWatch alarm uses the standard EC2 CPU utilization metric.

| Setting         | Value                 |
| --------------- | --------------------- |
| Namespace       | `AWS/EC2`             |
| Metric          | `CPUUtilization`      |
| Instance Name   | `aws-health-monitor`  |
| Instance ID     | `i-0cc3db6cd9ee5f449` |
| Statistic       | Average               |
| Period          | 1 minute              |
| Alarm Threshold | Greater than 5%       |

The threshold was lowered during testing to make it easier to demonstrate the alarm workflow. It is a test configuration, not a production recommendation.

## 3. CloudWatch Alarm

**Alarm name:** `aws-health-monitor-high-cpu`

The alarm evaluates the average CPU utilization metric against the configured threshold.

When the metric satisfies the alarm condition for the configured evaluation period, CloudWatch changes the alarm state to `ALARM`.

The main CloudWatch alarm states are:

- `OK` — the metric does not currently meet the alarm condition.
- `ALARM` — the metric meets the configured alarm condition.
- `INSUFFICIENT_DATA` — CloudWatch does not have enough data to evaluate the alarm.

## 4. SNS Notification Configuration

The alarm was connected to an SNS topic.

| Setting           | Value                         |
| ----------------- | ----------------------------- |
| SNS Topic         | `aws-health-monitor-alerts`   |
| Notification Type | Email                         |
| Trigger           | CloudWatch alarm state change |

The email subscription must be confirmed before SNS can deliver messages to the endpoint.

When the alarm enters `ALARM`, CloudWatch publishes a notification to the configured SNS topic, provided the alarm action is enabled and correctly configured.

## 5. Testing Procedure

The following procedure was used to test the monitoring configuration.

1. Connected to the Ubuntu EC2 instance using SSH.
2. Generated CPU load on the instance.
3. Inspected CPU activity using Linux `top`.
4. Opened CloudWatch and reviewed the EC2 `CPUUtilization` metric.
5. Adjusted the alarm threshold to 5% for the lab demonstration.
6. Observed the CloudWatch alarm enter the `ALARM` state.

**Test result:** The CloudWatch alarm successfully entered `ALARM`.

Email delivery should be recorded as successful only after the SNS notification is received and verified.

## 6. Linux and CloudWatch Observations

During testing, Linux `top` showed approximately 90% CPU steal time, while CloudWatch CPU utilization remained around 10%.

Useful commands for investigating this behaviour include:

```bash
nproc
uptime
top -bn1 | head -5
ps -eo pid,comm,%cpu --sort=-%cpu | head -10
```

These commands help inspect the number of available logical CPUs, load average, CPU statistics, and processes consuming CPU.

CPU steal time and EC2 `CPUUtilization` are different measurements. A high Linux load average does not necessarily mean that CloudWatch CPU utilization will exceed the same numerical value.

## 7. Troubleshooting

If the alarm does not enter `ALARM`, check:

- The selected EC2 instance and `CPUUtilization` metric.
- The threshold and comparison operator.
- The statistic and metric period.
- The evaluation period and datapoints-to-alarm settings.
- Whether recent metric data is available.
- Whether the alarm action is enabled.

If the alarm enters `ALARM` but no email arrives, check:

- Whether the SNS topic is configured as the alarm action.
- Whether the email subscription is confirmed.
- Whether the correct email endpoint was configured.
- The SNS topic's subscription status.
- The inbox spam or junk folder.

## 8. Cleanup

After testing:

1. Stop the CPU load processes.
2. Verify that the instance has returned to a stable state.
3. Confirm whether the alarm returns to `OK` once sufficient new metric data satisfies its recovery condition.
4. Remove unused lab resources when the project evidence has been collected.

## 9. Related Documentation

- [Project Overview](../README.md)
- [Architecture](../architecture/architecture.md)
- [Deployment Guide](../docs/deployment.md)
- [Troubleshooting Notes](../docs/troubleshooting.md)
