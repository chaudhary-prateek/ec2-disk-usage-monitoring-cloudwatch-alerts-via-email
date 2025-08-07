````markdown
# 📊 EC2 Disk Usage Monitoring with Amazon CloudWatch Agent

This guide describes how to manually install and configure the **CloudWatch Agent** on an **Ubuntu EC2 instance** to monitor **disk usage** and set up **email alerts** using CloudWatch Alarms.

---

## ✅ Prerequisites

- EC2 instance (Ubuntu)
- IAM role attached to the instance with the following policies:
  - `AmazonSSMManagedInstanceCore`
  - `CloudWatchAgentServerPolicy`
- Sudo privileges on the instance
- Access to AWS CloudWatch and SNS for setting up alerts

---

## 🔐 Step 1: Attach IAM Role to EC2

1. Go to **EC2 → Instances → [Your Instance]**
2. Click **Actions → Security → Modify IAM Role**
3. Select a role that has:
   - `AmazonSSMManagedInstanceCore`
   - `CloudWatchAgentServerPolicy`
4. Click **Update IAM Role**

---

## 📦 Step 2: Install the CloudWatch Agent

```bash
cd /tmp
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
````

---

## ⚙️ Step 3: Create Agent Config File

Create the config JSON file to monitor disk usage:

```bash
sudo nano /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

Paste the following content:

```json
{
  "metrics": {
    "append_dimensions": {
      "InstanceId": "${aws:InstanceId}"
    },
    "metrics_collected": {
      "disk": {
        "measurement": [
          "used_percent"
        ],
        "metrics_collection_interval": 60,
        "resources": [
          "/"
        ]
      }
    }
  }
}
```

Save and exit.

---

## ▶️ Step 4: Start CloudWatch Agent

Run the following command to start the agent using the above config:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s
```

---

## 🧹 Step 5: Clean Up Installer (Optional)

```bash
rm /tmp/amazon-cloudwatch-agent.deb
```

---

## 🔍 Step 6: Verify CloudWatch Metrics

1. Go to **CloudWatch → Metrics**
2. Navigate to `CWAgent` namespace
3. Check for the metric: `disk_used_percent` for your instance
4. Make sure values are being recorded

---

## 🔔 Step 7: Create Storage Alert (Alarm)

1. Go to **CloudWatch → Alarms → Create Alarm**
2. Choose metric: `CWAgent → disk_used_percent`
3. Set condition:

   * **Threshold**: Greater than `70`
   * **Evaluation period**: 1 out of 1 datapoints
4. Set Notification:

   * Choose or create an **SNS topic**
   * Add your email address and confirm it
5. Name the alarm and click **Create Alarm**

---

## ✅ Summary

| Step | Description                               |
| ---- | ----------------------------------------- |
| 1    | Attach IAM role to instance               |
| 2    | Install CloudWatch Agent                  |
| 3    | Create config file to monitor disk usage  |
| 4    | Start CloudWatch Agent with config        |
| 5    | (Optional) Remove installer               |
| 6    | Check CloudWatch metrics                  |
| 7    | Create alarm to notify on high disk usage |

---

## 🛠️ Useful Commands

Check disk usage:

```bash
df -h
```

Restart CloudWatch Agent:

```bash
sudo systemctl restart amazon-cloudwatch-agent
```

Stop Agent:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a stop
```

---

## 📬 Notification Tip

If using SNS for alerts, don’t forget to confirm the email address via the subscription confirmation email from AWS!

```
