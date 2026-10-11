# Troubleshooting Guide — AWS EC2 CPU Server Health Monitor

## 1. Overview

This document records common issues encountered while deploying and testing the EC2 CPU Server Health Monitor lab.

The goal is to follow an SRE troubleshooting approach:

1. Observe the symptom.
2. Collect evidence.
3. Identify possible causes.
4. Apply a controlled fix.
5. Verify the result.

## 2. Issue: Unable to Connect to EC2 Using SSH

### Symptoms

The SSH client cannot connect to the EC2 instance, even though the instance is running.

### Investigation

Check the instance state and public IPv4 address in the EC2 console.

Review the security group's inbound rules. SSH requires TCP port 22 to be permitted from the connecting client's public IP address.

### Resolution

Add an inbound rule with these settings:

| Setting  | Value |
| -------- | ----- |
| Type     | SSH   |
| Protocol | TCP   |
| Port     | 22    |
| Source   | My IP |

Connect from Git Bash using the correct private key:

```bash
chmod 400 aws-health-monitor.pem
ssh -i aws-health-monitor.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Replace the placeholder with the instance's current public IP address.

### Lessons Learned

- A running instance does not guarantee network connectivity.
- Security groups control permitted inbound traffic.
- Restrict SSH access to trusted source addresses instead of opening it to everyone.
- The public IP may change after stopping and restarting an instance unless an Elastic IP is used.

## 3. Issue: High Linux Load Average

### Symptoms

During CPU load testing, the instance reported high load averages. One observation showed values around `13.19`, `10.84`, and `6.81`.

### Investigation

Run:

```bash
uptime
nproc
top
```

Inspect CPU-consuming processes:

```bash
ps -eo pid,comm,%cpu --sort=-%cpu | head -10
```

The test instance reported two logical CPUs. Several `yes` processes were consuming CPU resources during the test.

The `top` output also showed CPU steal time (`st`) near 90% in one observation.

### Interpretation

Load average represents the number of runnable or uninterruptible tasks averaged over time. It is not a direct percentage of CPU utilization.

CPU steal time represents time when a virtual CPU was ready to run but the hypervisor did not schedule it. High steal time can indicate contention for physical CPU resources, but the observation alone does not establish the precise cause.

### Resolution

Stop the specific test processes that were started for the experiment. Avoid launching multiple load generators on a small instance.

Recheck the system:

```bash
uptime
top
```

### Lessons Learned

- A high load average does not necessarily mean CPU utilization is equally high.
- CPU steal time is an important signal when troubleshooting virtual machines.
- Always use controlled workloads and clean them up after testing.

## 4. Issue: Linux CPU Observations Differ From CloudWatch

### Symptoms

Linux showed high load and CPU steal time, while the CloudWatch `CPUUtilization` metric was around 10% during one observation.

### Investigation

Check Linux's current CPU statistics:

```bash
top -bn1 | head -5
```

Review the EC2 metric in CloudWatch:

1. Open **CloudWatch → Metrics**.
2. Select the EC2 namespace.
3. Open the per-instance metrics.
4. Select `CPUUtilization` for the correct instance.
5. Check the graph's time range and metric period.

### Interpretation

These observations are not necessarily contradictory. Linux and CloudWatch present different measurements, and their collection intervals may not align.

The CloudWatch `CPUUtilization` metric reflects EC2 CPU utilization as defined by AWS. Linux tools expose additional details, including user time, system time, idle time, and steal time.

A short-lived workload may also be missed or smoothed by the selected CloudWatch time window.

### Resolution

- Compare observations from approximately the same time.
- Verify that the selected CloudWatch metric belongs to the correct instance.
- Check the graph's time range and statistic.
- Use Linux process and CPU statistics alongside CloudWatch rather than treating either as the complete picture.

### Lessons Learned

Monitoring tools provide different perspectives. Reliable troubleshooting combines infrastructure metrics with operating-system evidence.

## 5. Issue: CloudWatch Alarm Does Not Enter ALARM

### Symptoms

The alarm remains in `OK` even when a CPU workload is running.

### Investigation

Review the alarm configuration:

- Metric namespace: `AWS/EC2`
- Metric: `CPUUtilization`
- Correct EC2 instance selected
- Statistic and period
- Threshold and comparison operator
- Evaluation periods and datapoints-to-alarm
- Alarm state and metric graph

Confirm that CloudWatch has recent data for the selected instance.

### Resolution

For a demonstration, a low threshold such as greater than 5% can make the alarm easier to trigger. Ensure that the statistic, period, and evaluation settings match the documented configuration.

Wait for sufficient metric data and alarm evaluation. Do not assume that starting a CPU process will immediately change the alarm state.

### Verification

In this lab, the alarm `aws-health-monitor-high-cpu` entered `ALARM` after the threshold was lowered to 5%.

A production alarm should use a threshold and evaluation window appropriate to the workload and its operational requirements.

## 6. Issue: SNS Email Notification Is Not Received

### Symptoms

The alarm enters `ALARM`, but no email notification is visible.

### Investigation

Check the following:

1. Open the SNS topic `aws-health-monitor-alerts`.
2. Verify that the email subscription status is confirmed.
3. Open the CloudWatch alarm and review its notification action.
4. Confirm that the alarm action references the intended SNS topic.
5. Check the inbox, spam folder, and any email filtering rules.

### Resolution

Confirm the SNS subscription using the confirmation email. Correct the alarm action or subscription configuration if necessary.

### Verification

A successful alarm state transition does not by itself prove that an email was delivered. Record email notification testing as complete only after the message is received.

## 7. Operational Checklist

Before closing a CPU monitoring incident, verify:

- [ ] EC2 instance state and connectivity
- [ ] Current CPU utilization and load average
- [ ] Top CPU-consuming processes
- [ ] CPU steal time, if applicable
- [ ] Correct CloudWatch instance metric
- [ ] Alarm threshold and evaluation settings
- [ ] SNS subscription and alarm action
- [ ] Test workload stopped
- [ ] Evidence captured for the incident record

## 8. SRE Takeaways

This lab demonstrates several practical SRE principles:

- Validate symptoms using multiple independent sources.
- Separate observed facts from suspected causes.
- Use monitoring thresholds that reflect operational needs.
- Verify alert delivery rather than assuming that configuration guarantees success.
- Keep test workloads controlled and clean up resources after validation.
- Document the evidence, resolution, and lessons learned for future incidents.
