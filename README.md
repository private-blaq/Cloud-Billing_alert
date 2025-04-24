# Automating AWS Billing Alarms with EC2

This project automates the creation of billing alarms on AWS using an EC2 instance and Python. It helps cloud engineers and system administrators monitor cloud costs and avoid surprise billing.

---

## 🚀 How It Works

> This project runs a Python script inside an EC2 instance with proper IAM permissions to create CloudWatch billing alarms and SNS notifications.

---

## 📁 Project Structure

```
├── billing_alarm.py         # Python script that creates SNS topic and CloudWatch billing alarms
├── README.md                # Documentation
```

---

## 💡 High-Level Steps

1. Launch an EC2 instance (Amazon Linux 2 preferred).
2. Attach an IAM role with permissions for CloudWatch and SNS.
3. SSH into the instance.
4. Run the Python script to:
   - Create an SNS topic and email subscription
   - Set up 3 billing alarms (for example at $10, $20, $30)

---

## 🔐 Step 1: IAM Role with Required Permissions

Create a role with this inline policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudwatch:PutMetricAlarm",
        "cloudwatch:DescribeAlarms",
        "sns:CreateTopic",
        "sns:Subscribe",
        "sns:Publish"
      ],
      "Resource": "*"
    }
  ]
}
```

> Attach this role to your EC2 when launching it.

---

## 🖥️ Step 2: Launch EC2 & SSH

Use Amazon Linux 2 or Ubuntu. Once launched and SSH’d into the instance:

```bash
sudo yum update -y
sudo yum install python3 -y
pip3 install boto3
```

---

## 🐍 Step 3: Python Script to Create Billing Alarms

Create a file `billing_alarm.py`:

```python
import boto3

# Set up clients
sns = boto3.client("sns", region_name="us-east-1")
cloudwatch = boto3.client("cloudwatch", region_name="us-east-1")

# 1. Create SNS Topic
topic = sns.create_topic(Name="billing-alerts-topic")
topic_arn = topic["TopicArn"]

# 2. Subscribe email
email = "youremail@example.com"  # replace
sns.subscribe(TopicArn=topic_arn, Protocol="email", Endpoint=email)
print(f"Check your email ({email}) to confirm SNS subscription.")

# 3. Define thresholds
thresholds = [10, 20, 30]

for value in thresholds:
    alarm_name = f"billing-alarm-{value}"
    cloudwatch.put_metric_alarm(
        AlarmName=alarm_name,
        ComparisonOperator="GreaterThanThreshold",
        EvaluationPeriods=1,
        MetricName="EstimatedCharges",
        Namespace="AWS/Billing",
        Period=21600,
        Statistic="Maximum",
        Threshold=value,
        ActionsEnabled=True,
        AlarmActions=[topic_arn],
        AlarmDescription=f"Billing alarm for charges > ${value}",
        Dimensions=[
            {"Name": "Currency", "Value": "USD"}
        ],
        TreatMissingData="notBreaching"
    )

print("Billing alarms created.")
```

---

## ▶️ Step 4: Run the Script

```bash
python3 billing_alarm.py
```

Then:
- ✅ Go to your email and confirm the SNS subscription.
- ✅ Check CloudWatch → Alarms to see the 3 billing alarms.

---

## 🛠 Bonus: Automate with EC2 User Data

To run this script automatically at instance launch, embed it into **User Data**:

```bash
#!/bin/bash
sudo yum update -y
sudo yum install python3 -y
pip3 install boto3

cat <<EOF > /home/ec2-user/billing_alarm.py
<your python script goes here>
EOF

python3 /home/ec2-user/billing_alarm.py
```

---

## 📄 License

This project is licensed under the MIT License.

---

## 🙋 Author

**Adedia Michael**  
Cloud Enthusiast | DevOps Learner | Trainer
