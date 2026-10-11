# Deployment Guide — AWS EC2 CPU Server Health Monitor

## 1. Objective

This guide explains how to deploy a basic EC2 CPU monitoring and alerting solution using AWS services.

The lab uses Amazon EC2 to host an Ubuntu Linux instance, Amazon CloudWatch to monitor CPU utilization, and Amazon SNS to deliver alarm notifications through email.

## 2. Prerequisites

Before starting, ensure you have:

- An AWS account with permissions to create and manage EC2, CloudWatch, and SNS resources.
- An SSH client such as Git Bash on Windows.
- An EC2 key pair for SSH authentication.
- An email address that you can access to confirm the SNS subscription.
- An understanding of basic Linux commands.

**Cost and security note:** AWS resources may incur charges. Review current pricing, use a small instance suitable for the exercise, restrict SSH access to your own IP, and remove resources when finished.

## 3. Create the EC2 Instance

1. Open the AWS Management Console.
2. Navigate to **EC2 → Instances → Launch instances**.
3. Enter the instance name:

   `aws-health-monitor`

4. Select an Ubuntu Server AMI.
5. Select a small instance type appropriate for the lab, such as `t3.micro`, subject to account availability and pricing.
6. Create or select an EC2 key pair and securely store the private key.
7. Configure the security group to allow SSH on TCP port 22 **only from your current public IP address**.
8. Launch the instance.

Wait until the instance is running and its status checks have passed.

## 4. Connect to the Instance Using SSH

In Git Bash, move to the directory containing your private key.

Restrict access to the private key file:

```bash
chmod 400 aws-health-monitor.pem
```

Connect using the instance's public IPv4 address:

```bash
ssh -i aws-health-monitor.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Replace `YOUR_EC2_PUBLIC_IP` with the address shown in the EC2 console.

If the connection fails, verify the public IP, instance status, security-group inbound rules, route to the internet, and private-key permissions.

Never commit the private key to GitHub.

## 5. Inspect Linux CPU Information

After connecting, run the following commands:

```bash
nproc
uptime
top
```

To display the highest CPU-consuming processes:

```bash
ps -eo pid,comm,%cpu --sort=-%cpu | head -10
```

These commands help inspect logical CPUs, load average, CPU statistics, and process-level resource consumption.

Linux CPU statistics are useful for troubleshooting, but they are not identical to CloudWatch metrics.

## 6. Verify the CloudWatch Metric

1. Open **CloudWatch** in the AWS Management Console.
2. Navigate to **Metrics → All metrics**.
3. Select the EC2 namespace and locate the per-instance metrics.
4. Find `CPUUtilization` for the `aws-health-monitor` instance.
5. Select the metric and review its graph.

EC2 standard metrics are collected automatically. Basic monitoring generally provides five-minute metric intervals; detailed monitoring can provide one-minute intervals and may incur additional charges. Confirm the monitoring configuration for your instance.

## 7. Create an SNS Topic

1. Open **Amazon SNS → Topics**.
2. Choose **Create topic**.
3. Select the Standard topic type.
4. Enter the topic name:

   `aws-health-monitor-alerts`

5. Create the topic.

### Create an email subscription

1. Open the new SNS topic.
2. Choose **Create subscription**.
3. Select `Email` as the protocol.
4. Enter an email address you can access.
5. Create the subscription.
6. Open the confirmation email and confirm the subscription.

Verify that the subscription status is confirmed before testing notifications.

## 8. Create the CloudWatch Alarm

1. Open **CloudWatch → Alarms → All alarms**.
2. Choose **Create alarm**.
3. Select the EC2 `CPUUtilization` metric for the correct instance.
4. Configure the statistic, period, threshold, and evaluation settings.
5. For the demonstration, use a threshold greater than 5% if you want to reproduce the test configuration.
6. Configure the alarm action to publish a notification to `aws-health-monitor-alerts` when the alarm enters `ALARM`.
7. Name the alarm:

   `aws-health-monitor-high-cpu`

8. Review the settings and create the alarm.

The 5% threshold is deliberately low for demonstration purposes. Choose a suitable threshold and evaluation window for real workloads rather than using this test value in production.

## 9. Test the Alarm

Use CloudWatch's metric graph to observe CPU utilization. If you choose to generate a controlled CPU workload on the instance, monitor it closely and stop the workload after testing.

For a temporary Linux CPU load test, one option is:

```bash
yes > /dev/null &
```

This starts one CPU-consuming process. Record its process ID and stop only that process when the test is complete:

```bash
kill PID
```

Replace `PID` with the process ID returned by the shell or found using `ps`.

Do not launch many load processes on a small instance. CloudWatch uses metric sampling and alarm evaluation windows, so a short workload may not immediately cause an alarm transition.

### Verify the results

- Confirm that CloudWatch has received metric data.
- Observe whether the alarm enters `ALARM`.
- Check the alarm's configured SNS action.
- Verify that the SNS email subscription is confirmed.
- Check the email inbox and spam folder for the notification.

In the original lab, the alarm entered `ALARM` after the threshold was lowered to 5%. Email receipt must be verified separately.

## 10. Troubleshooting

| Problem                                              | What to check                                                            |
| ---------------------------------------------------- | ------------------------------------------------------------------------ |
| SSH connection fails                                 | Public IP, key permissions, security group, instance status              |
| CPU metric is missing                                | Correct instance, metric namespace, selected time range                  |
| Alarm stays in `OK`                                  | Threshold, evaluation window, metric data                                |
| Alarm shows `INSUFFICIENT_DATA`                      | Metric availability and alarm configuration                              |
| Email is not received                                | SNS subscription confirmation, alarm action, spam folder                 |
| Linux load appears high but CloudWatch CPU looks low | Compare CPU steal time, load average, and CloudWatch's metric definition |

## 11. Cleanup

After completing the exercise:

1. Stop all test workload processes.
2. Confirm that the instance is stable.
3. Save screenshots or notes required for your portfolio.
4. Delete the CloudWatch alarm if it is no longer needed.
5. Delete the SNS topic and subscription if they are no longer needed.
6. Terminate the EC2 instance if you do not intend to reuse it.

Check for other billable resources before finishing.

## 12. Expected Outcome

After completing this lab, you should be able to:

- Launch and access an Ubuntu EC2 instance.
- Inspect CPU behaviour using Linux commands.
- Locate standard EC2 CPU metrics in CloudWatch.
- Configure a CloudWatch alarm.
- Connect the alarm to an SNS email notification.
- Investigate differences between Linux CPU observations and CloudWatch metrics.

## 13. Related Documentation

- [Project Overview](../README.md)
- [Architecture](../architecture/architecture.md)
- [CloudWatch Alarm Configuration](../cloudwatch/alarms.md)
- [Troubleshooting Notes](troubleshooting.md)
