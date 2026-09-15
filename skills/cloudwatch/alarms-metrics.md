# CloudWatch Alarms and Metrics

Detailed patterns for CloudWatch alarms and custom metrics.

## Alarm Types

### Standard Metric Alarm

Triggers based on a single metric threshold:

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "HighCPUUtilization" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

### Metric Math Alarm

Combines multiple metrics with expressions:

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "HighErrorRate" \
  --metrics '[
    {
      "Id": "e1",
      "MetricStat": {
        "Metric": {
          "Namespace": "AWS/ApplicationELB",
          "MetricName": "HTTPCode_ELB_5XX_Count",
          "Dimensions": [{"Name": "LoadBalancer", "Value": "app/my-lb/1234567890123456"}]
        },
        "Period": 60,
        "Stat": "Sum"
      },
      "ReturnData": false
    },
    {
      "Id": "e2",
      "MetricStat": {
        "Metric": {
          "Namespace": "AWS/ApplicationELB",
          "MetricName": "RequestCount",
          "Dimensions": [{"Name": "LoadBalancer", "Value": "app/my-lb/1234567890123456"}]
        },
        "Period": 60,
        "Stat": "Sum"
      },
      "ReturnData": false
    },
    {
      "Id": "e3",
      "Expression": "IF(e2>0, e1/e2*100, 0)",
      "Label": "Error Rate %",
      "ReturnData": true
    }
  ]' \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 3 \
  --datapoints-to-alarm 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

### Anomaly Detection Alarm

Uses machine learning to detect anomalies:

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "AnomalousLatency" \
  --metrics '[
    {
      "Id": "m1",
      "MetricStat": {
        "Metric": {
          "Namespace": "AWS/Lambda",
          "MetricName": "Duration",
          "Dimensions": [{"Name": "FunctionName", "Value": "MyFunction"}]
        },
        "Period": 60,
        "Stat": "Average"
      },
      "ReturnData": true
    },
    {
      "Id": "ad1",
      "Expression": "ANOMALY_DETECTION_BAND(m1, 2)",
      "Label": "AnomalyDetectionBand",
      "ReturnData": true
    }
  ]' \
  --threshold-metric-id ad1 \
  --comparison-operator LessThanLowerOrGreaterThanUpperThreshold \
  --evaluation-periods 3 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

### Composite Alarm

Combines multiple alarms with boolean logic:

```bash
# Create component alarms first
aws cloudwatch put-metric-alarm \
  --alarm-name "HighCPU" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0

aws cloudwatch put-metric-alarm \
  --alarm-name "HighMemory" \
  --metric-name MemoryUtilization \
  --namespace CWAgent \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0

# Create composite alarm
aws cloudwatch put-composite-alarm \
  --alarm-name "InstanceUnhealthy" \
  --alarm-rule "ALARM(HighCPU) AND ALARM(HighMemory)" \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

### Log Alarm (Multi-Contributor)

Evaluates a scheduled Logs Insights query. A `by` clause in `AggregationExpression` tracks each group as a contributor; the alarm is in ALARM if any contributor breaches. SNS/Lambda actions fire per contributor.

```bash
aws cloudwatch put-log-alarm \
  --alarm-name "EndpointLatency" \
  --comparison-operator GreaterThanThreshold \
  --threshold 2000 \
  --query-results-to-evaluate 3 \
  --query-results-to-alarm 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts \
  --scheduled-query-configuration '{
    "QueryString": "fields endpoint, latency | filter ispresent(latency)",
    "LogGroupIdentifiers": ["/app/api"],
    "ScheduledQueryRoleARN": "arn:aws:iam::123456789012:role/ScheduledQueryRole",
    "AggregationExpression": "avg(latency) by endpoint | sort desc",
    "ScheduleConfiguration": {
      "ScheduleExpression": "rate(5 minutes)",
      "StartTimeOffset": 420,
      "EndTimeOffset": 120
    }
  }'

# Which contributors are in ALARM
aws cloudwatch describe-alarm-contributors --alarm-name EndpointLatency
```

- Aggregation functions: `count(*)`, `avg`, `sum`, `min`, `max`; `bin()` not allowed in the `by` clause
- Limits: 5 `by` fields, 500 contributors returned per run (sort by value so the worst are kept; `EvaluationState` = `PARTIAL_DATA` when exceeded), 100 contributors in ALARM
- `--action-log-line-count` (0-50) adds raw log lines to SNS email notifications; requires `--action-log-line-role-arn` (role trusting `cloudwatch.amazonaws.com` with `logs:GetQueryResults`)
- `EndTimeOffset` shifts the window back to allow for ingestion delay

### Wall Clock Window and Warm-Up

```bash
# Daily backup check aligned to local midnight, quiet for 60 min after create/update
aws cloudwatch put-metric-alarm \
  --alarm-name "DailyBackupMissing" \
  --namespace MyApp \
  --metric-name BackupsCompleted \
  --statistic Sum \
  --period 86400 \
  --evaluation-periods 1 \
  --threshold 1 \
  --comparison-operator LessThanThreshold \
  --treat-missing-data breaching \
  --evaluation-window 'WallClockWindow={Timezone=America/New_York}' \
  --warm-up-configuration WarmUpPeriodDurationInMinutes=60 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

- Wall clock windows support periods 60, 300, 3600, 86400, 604800 only; not PromQL alarms; data is reflected only after the clock period ends
- Metrics Insights alarms with a wall clock window: Period x (EvaluationPeriods + 1) must be <= 3 hours
- Warm-up (1-2880 min) applies to metric and log alarms; alarm stays INSUFFICIENT_DATA with no actions. Ends early once data fills the window unless `OnlyStartEvaluatingAfterWarmUpPeriodEnds=true`

## Common Alarm Patterns

### Lambda Function Health

```bash
# Errors alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "Lambda-MyFunction-Errors" \
  --metric-name Errors \
  --namespace AWS/Lambda \
  --statistic Sum \
  --period 60 \
  --threshold 5 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --dimensions Name=FunctionName,Value=MyFunction \
  --treat-missing-data notBreaching \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts

# Throttles alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "Lambda-MyFunction-Throttles" \
  --metric-name Throttles \
  --namespace AWS/Lambda \
  --statistic Sum \
  --period 60 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --dimensions Name=FunctionName,Value=MyFunction \
  --treat-missing-data notBreaching \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts

# Duration alarm (approaching timeout)
aws cloudwatch put-metric-alarm \
  --alarm-name "Lambda-MyFunction-Duration" \
  --metric-name Duration \
  --namespace AWS/Lambda \
  --extended-statistic p99 \
  --period 300 \
  --threshold 25000 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=FunctionName,Value=MyFunction \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

### DynamoDB Table Health

```bash
# Consumed capacity alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "DynamoDB-MyTable-HighReadCapacity" \
  --metric-name ConsumedReadCapacityUnits \
  --namespace AWS/DynamoDB \
  --statistic Average \
  --period 60 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 5 \
  --dimensions Name=TableName,Value=MyTable \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts

# Throttled requests
aws cloudwatch put-metric-alarm \
  --alarm-name "DynamoDB-MyTable-Throttled" \
  --metric-name ThrottledRequests \
  --namespace AWS/DynamoDB \
  --statistic Sum \
  --period 60 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --dimensions Name=TableName,Value=MyTable \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

### API Gateway Health

```bash
# 5XX errors
aws cloudwatch put-metric-alarm \
  --alarm-name "APIGW-MyAPI-5XXErrors" \
  --metric-name 5XXError \
  --namespace AWS/ApiGateway \
  --statistic Sum \
  --period 60 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=ApiName,Value=MyAPI \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts

# Latency P99
aws cloudwatch put-metric-alarm \
  --alarm-name "APIGW-MyAPI-HighLatency" \
  --metric-name Latency \
  --namespace AWS/ApiGateway \
  --extended-statistic p99 \
  --period 300 \
  --threshold 5000 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 3 \
  --dimensions Name=ApiName,Value=MyAPI \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

### SQS Queue Health

```bash
# Messages in queue (backlog)
aws cloudwatch put-metric-alarm \
  --alarm-name "SQS-MyQueue-Backlog" \
  --metric-name ApproximateNumberOfMessagesVisible \
  --namespace AWS/SQS \
  --statistic Average \
  --period 300 \
  --threshold 1000 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 3 \
  --dimensions Name=QueueName,Value=MyQueue \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts

# Age of oldest message
aws cloudwatch put-metric-alarm \
  --alarm-name "SQS-MyQueue-OldMessages" \
  --metric-name ApproximateAgeOfOldestMessage \
  --namespace AWS/SQS \
  --statistic Maximum \
  --period 300 \
  --threshold 3600 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=QueueName,Value=MyQueue \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

## Custom Metrics

### High-Resolution Metrics

Publish metrics with 1-second resolution:

```python
import boto3

cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[
        {
            'MetricName': 'RequestLatency',
            'Value': 125.5,
            'Unit': 'Milliseconds',
            'StorageResolution': 1,  # 1 second resolution
            'Dimensions': [
                {'Name': 'Endpoint', 'Value': '/api/orders'}
            ]
        }
    ]
)
```

### Batched Metric Publishing

```python
import boto3
from datetime import datetime

cloudwatch = boto3.client('cloudwatch')

# Collect metrics
metric_data = [
    {
        'MetricName': 'OrdersProcessed',
        'Value': 42,
        'Unit': 'Count',
        'Timestamp': datetime.utcnow(),
        'Dimensions': [
            {'Name': 'Environment', 'Value': 'Production'}
        ]
    },
    {
        'MetricName': 'ProcessingTime',
        'Value': 250.5,
        'Unit': 'Milliseconds',
        'Timestamp': datetime.utcnow(),
        'Dimensions': [
            {'Name': 'Environment', 'Value': 'Production'}
        ]
    }
]

# Publish in batch (max 1000 per call)
cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=metric_data
)
```

### Embedded Metric Format (EMF)

Publish metrics directly from Lambda logs:

```python
import json

def handler(event, context):
    # Process request
    processing_time = 125.5

    # Emit EMF log
    print(json.dumps({
        "_aws": {
            "Timestamp": int(time.time() * 1000),
            "CloudWatchMetrics": [{
                "Namespace": "MyApp",
                "Dimensions": [["Environment", "Endpoint"]],
                "Metrics": [
                    {"Name": "RequestLatency", "Unit": "Milliseconds"},
                    {"Name": "RequestCount", "Unit": "Count"}
                ]
            }]
        },
        "Environment": "Production",
        "Endpoint": "/api/orders",
        "RequestLatency": processing_time,
        "RequestCount": 1
    }))

    return {"statusCode": 200}
```

## Metric Math Functions

| Function | Description | Example |
|----------|-------------|---------|
| `SUM` | Sum of metrics | `SUM([m1, m2, m3])` |
| `AVG` | Average | `AVG([m1, m2])` |
| `MIN`, `MAX` | Minimum/Maximum | `MAX([m1, m2])` |
| `RATE` | Per-second rate of change | `RATE(m1)` |
| `PERIOD` | Current period in seconds | `m1 / PERIOD(m1)` |
| `FILL` | Replace missing data | `FILL(m1, 0)` |
| `IF` | Conditional | `IF(m1 > 100, m1, 0)` |
| `ANOMALY_DETECTION_BAND` | ML-based band | `ANOMALY_DETECTION_BAND(m1, 2)` |
| `SEARCH` | Dynamic metrics | `SEARCH('{AWS/EC2,InstanceId} MetricName="CPUUtilization"', 'Average', 300)` |

### Example: Percentage Calculation

```
e1 = errors metric
e2 = requests metric
errorRate = IF(e2 > 0, (e1 / e2) * 100, 0)
```

### Example: Aggregate Across Dimensions

```
SEARCH('{AWS/Lambda,FunctionName} MetricName="Errors"', 'Sum', 60)
```

## Alarm Actions

### SNS Notification

```bash
--alarm-actions arn:aws:sns:us-east-1:123456789012:my-topic
```

### Auto Scaling

```bash
--alarm-actions arn:aws:autoscaling:us-east-1:123456789012:scalingPolicy:12345678-1234-1234-1234-123456789012:autoScalingGroupName/my-asg:policyName/scale-out
```

### EC2 Actions

```bash
# Stop instance
--alarm-actions arn:aws:automate:us-east-1:ec2:stop

# Terminate instance
--alarm-actions arn:aws:automate:us-east-1:ec2:terminate

# Recover instance
--alarm-actions arn:aws:automate:us-east-1:ec2:recover
```

### Lambda Function

Alarms invoke Lambda directly (function, version, or alias ARN). Grant the alarm permission first:

```bash
aws lambda add-permission \
  --function-name HandleAlarm \
  --statement-id AlarmAction \
  --action lambda:InvokeFunction \
  --principal lambda.alarms.cloudwatch.amazonaws.com \
  --source-account 123456789012 \
  --source-arn arn:aws:cloudwatch:us-east-1:123456789012:alarm:HighCPU

# Then on the alarm:
--alarm-actions arn:aws:lambda:us-east-1:123456789012:function:HandleAlarm
```

## Alarm Mute Rules

Mute actions (all states) during scheduled windows; alarms keep evaluating and EventBridge events still emit. Up to 100 alarms per rule.

```bash
# Mute every Sunday 02:00 for 4 hours
aws cloudwatch put-alarm-mute-rule \
  --name weekly-maintenance \
  --rule 'Schedule={Expression=cron(0 2 * * SUN),Duration=PT4H,Timezone=America/New_York}' \
  --mute-targets AlarmNames=HighCPU,HighMemory

# One-time window: Expression=at(2026-12-23T00:00),Duration=P7D
aws cloudwatch list-alarm-mute-rules --statuses ACTIVE
aws cloudwatch delete-alarm-mute-rule --alarm-mute-rule-name weekly-maintenance
```

- When the window ends (or the rule is deleted/updated or the alarm removed from targets), muted actions run if the alarm is still in the state it was muted in
- `enable-alarm-actions` does not unmute; `disable-alarm-actions` is permanent until re-enabled
- IAM: `cloudwatch:PutAlarmMuteRule` is needed on the mute rule resource and on each targeted alarm
