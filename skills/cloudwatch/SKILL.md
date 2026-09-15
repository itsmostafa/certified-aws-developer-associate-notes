---
name: cloudwatch
description: AWS CloudWatch monitoring for logs, metrics, alarms, and dashboards. Use when setting up monitoring, creating alarms, querying logs with Insights, configuring metric filters, building dashboards, or troubleshooting application issues.
last_updated: "2026-09-14"
doc_source: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/
---

# AWS CloudWatch

Amazon CloudWatch provides monitoring and observability for AWS resources and applications. It collects metrics, logs, and events, enabling you to monitor, troubleshoot, and optimize your AWS environment.

## Table of Contents

- [Core Concepts](#core-concepts)
- [Common Patterns](#common-patterns)
- [CLI Reference](#cli-reference)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [References](#references)

## Core Concepts

### Metrics

Time-ordered data points published to CloudWatch. Key components:
- **Namespace**: Container for metrics (e.g., `AWS/Lambda`)
- **Metric name**: Name of the measurement (e.g., `Invocations`)
- **Dimensions**: Name-value pairs for filtering (e.g., `FunctionName=MyFunc`)
- **Statistics**: Aggregations (Sum, Average, Min, Max, SampleCount, pN)

### Logs

Log data from AWS services and applications:
- **Log groups**: Collections of log streams
- **Log streams**: Sequences of log events from same source
- **Log events**: Individual log entries with timestamp and message

### Alarms

Automated actions based on metric thresholds:
- **States**: OK, ALARM, INSUFFICIENT_DATA
- **Actions**: SNS notifications, Lambda functions, Auto Scaling, EC2 actions, Systems Manager OpsItems
- **Types**: metric (incl. metric math, anomaly detection, Metrics Insights), composite, log (scheduled Logs Insights query evaluated M-out-of-N on query results)
- **Mute rules**: scheduled windows that mute actions while alarms keep evaluating

## Common Patterns

### Create a Metric Alarm

**AWS CLI:**

```bash
# CPU utilization alarm for EC2
aws cloudwatch put-metric-alarm \
  --alarm-name "HighCPU-i-1234567890abcdef0" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts \
  --ok-actions arn:aws:sns:us-east-1:123456789012:alerts
```

**boto3:**

```python
import boto3

cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_alarm(
    AlarmName='HighCPU-i-1234567890abcdef0',
    MetricName='CPUUtilization',
    Namespace='AWS/EC2',
    Statistic='Average',
    Period=300,
    Threshold=80.0,
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=2,
    Dimensions=[
        {'Name': 'InstanceId', 'Value': 'i-1234567890abcdef0'}
    ],
    AlarmActions=['arn:aws:sns:us-east-1:123456789012:alerts'],
    OKActions=['arn:aws:sns:us-east-1:123456789012:alerts']
)
```

### Lambda Error Rate Alarm

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "LambdaErrorRate-MyFunction" \
  --metrics '[
    {
      "Id": "errors",
      "MetricStat": {
        "Metric": {
          "Namespace": "AWS/Lambda",
          "MetricName": "Errors",
          "Dimensions": [{"Name": "FunctionName", "Value": "MyFunction"}]
        },
        "Period": 60,
        "Stat": "Sum"
      },
      "ReturnData": false
    },
    {
      "Id": "invocations",
      "MetricStat": {
        "Metric": {
          "Namespace": "AWS/Lambda",
          "MetricName": "Invocations",
          "Dimensions": [{"Name": "FunctionName", "Value": "MyFunction"}]
        },
        "Period": 60,
        "Stat": "Sum"
      },
      "ReturnData": false
    },
    {
      "Id": "errorRate",
      "Expression": "errors/invocations*100",
      "Label": "Error Rate",
      "ReturnData": true
    }
  ]' \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 3 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

### Query Logs with Insights

```bash
# Find errors in Lambda logs
aws logs start-query \
  --log-group-name /aws/lambda/MyFunction \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string '
    fields @timestamp, @message
    | filter @message like /ERROR/
    | sort @timestamp desc
    | limit 50
  '

# Get query results
aws logs get-query-results --query-id <query-id>
```

**boto3:**

```python
import boto3
import time

logs = boto3.client('logs')

# Start query
response = logs.start_query(
    logGroupName='/aws/lambda/MyFunction',
    startTime=int(time.time()) - 3600,
    endTime=int(time.time()),
    queryString='''
        fields @timestamp, @message
        | filter @message like /ERROR/
        | sort @timestamp desc
        | limit 50
    '''
)

query_id = response['queryId']

# Wait for results
while True:
    result = logs.get_query_results(queryId=query_id)
    if result['status'] == 'Complete':
        break
    time.sleep(1)

for row in result['results']:
    print(row)
```

### Create Metric Filter

Extract metrics from log patterns:

```bash
# Create metric filter for error count
aws logs put-metric-filter \
  --log-group-name /aws/lambda/MyFunction \
  --filter-name ErrorCount \
  --filter-pattern "ERROR" \
  --metric-transformations \
    metricName=ErrorCount,metricNamespace=MyApp,metricValue=1,defaultValue=0
```

### Create a Log Alarm

Alarm directly on a Logs Insights query (no metric filter needed). CloudWatch creates and manages the underlying scheduled query. The `ScheduledQueryRoleARN` role must trust `logs.amazonaws.com` and allow `logs:StartQuery` and `logs:GetQueryResults` on the log group ARNs, plus `logs:StopQuery` and `logs:DescribeLogGroups` on `"Resource": "*"` (these two support no resource-level permissions, so a log-group-scoped statement never matches them).

```bash
# ALARM when >100 errors in 3 of the last 5 query runs
aws cloudwatch put-log-alarm \
  --alarm-name "HighErrorCount" \
  --comparison-operator GreaterThanThreshold \
  --threshold 100 \
  --query-results-to-evaluate 5 \
  --query-results-to-alarm 3 \
  --treat-missing-data notBreaching \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts \
  --scheduled-query-configuration '{
    "QueryString": "fields @timestamp, @message | filter @message like /ERROR/",
    "LogGroupIdentifiers": ["/aws/lambda/MyFunction"],
    "ScheduledQueryRoleARN": "arn:aws:iam::123456789012:role/ScheduledQueryRole",
    "AggregationExpression": "count(*)",
    "ScheduleConfiguration": {
      "ScheduleExpression": "rate(10 minutes)",
      "StartTimeOffset": 600
    }
  }'

# Log alarms are omitted from describe-alarms unless requested
aws cloudwatch describe-alarms --alarm-types LogAlarm
```

### Publish Custom Metrics

```python
import boto3

cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[
        {
            'MetricName': 'OrdersProcessed',
            'Value': 1,
            'Unit': 'Count',
            'Dimensions': [
                {'Name': 'Environment', 'Value': 'Production'},
                {'Name': 'OrderType', 'Value': 'Standard'}
            ]
        }
    ]
)
```

### Create Dashboard

```bash
cat > dashboard.json << 'EOF'
{
  "widgets": [
    {
      "type": "metric",
      "x": 0, "y": 0, "width": 12, "height": 6,
      "properties": {
        "title": "Lambda Invocations",
        "metrics": [
          ["AWS/Lambda", "Invocations", "FunctionName", "MyFunction"]
        ],
        "period": 60,
        "stat": "Sum",
        "region": "us-east-1"
      }
    },
    {
      "type": "log",
      "x": 12, "y": 0, "width": 12, "height": 6,
      "properties": {
        "title": "Recent Errors",
        "query": "SOURCE '/aws/lambda/MyFunction' | filter @message like /ERROR/ | limit 20",
        "region": "us-east-1"
      }
    }
  ]
}
EOF

aws cloudwatch put-dashboard \
  --dashboard-name MyAppDashboard \
  --dashboard-body file://dashboard.json
```

## CLI Reference

### Metrics Commands

| Command | Description |
|---------|-------------|
| `aws cloudwatch put-metric-data` | Publish custom metrics |
| `aws cloudwatch get-metric-data` | Retrieve metric values |
| `aws cloudwatch get-metric-statistics` | Get aggregated statistics |
| `aws cloudwatch list-metrics` | List available metrics |

### Alarms Commands

| Command | Description |
|---------|-------------|
| `aws cloudwatch put-metric-alarm` | Create or update alarm |
| `aws cloudwatch put-log-alarm` | Create or update log query alarm |
| `aws cloudwatch describe-alarms` | List alarms (`--alarm-types LogAlarm` for log alarms) |
| `aws cloudwatch describe-alarm-contributors` | Show breaching contributors of a multi-contributor alarm |
| `aws cloudwatch put-alarm-mute-rule` | Create or update scheduled mute window |
| `aws cloudwatch list-alarm-mute-rules` | List mute rules (`--statuses SCHEDULED ACTIVE EXPIRED`) |
| `aws cloudwatch delete-alarm-mute-rule` | Delete mute rule (unmutes immediately) |
| `aws cloudwatch set-alarm-state` | Manually set alarm state |
| `aws cloudwatch delete-alarms` | Delete alarms |

### Logs Commands

| Command | Description |
|---------|-------------|
| `aws logs create-log-group` | Create log group |
| `aws logs put-log-events` | Write log events |
| `aws logs filter-log-events` | Search log events |
| `aws logs start-query` | Start Insights query |
| `aws logs put-metric-filter` | Create metric filter |
| `aws logs put-retention-policy` | Set log retention |

## Best Practices

### Metrics

- **Use dimensions wisely** — too many creates metric explosion
- **Aggregate before publishing** — batch custom metrics
- **Use high-resolution metrics** (1-second) only when needed
- **Set meaningful units** for custom metrics

### Alarms

- **Use composite alarms** for complex conditions
- **Set appropriate evaluation periods** to avoid flapping
- **Include OK actions** to track recovery
- **Use anomaly detection** for dynamic thresholds
- **Use log alarms** instead of metric filter + metric alarm for query-based conditions; set `--treat-missing-data notBreaching` for sparse errors, `breaching` to detect logs that stop arriving
- **Stagger log alarm schedules** — concurrent scheduled query executions per account are capped at 100
- **Set `--warm-up-configuration`** on alarms created alongside new resources (1-2880 min) to avoid noise before metrics publish
- **Use `--evaluation-window WallClockWindow={Timezone=...}`** for daily/weekly batch or backup alarms; keep the default sliding window for Auto Scaling
- **Use mute rules for maintenance**, not `disable-alarm-actions`; `enable-alarm-actions` does not unmute an active mute rule

### Logs

- **Set retention policies** — don't keep logs forever
- **Use structured logging** (JSON) for better querying
- **Create metric filters** for key events
- **Use Contributor Insights** for top-N analysis

### Cost Optimization

- **Delete unused dashboards**
- **Reduce log retention** for non-critical logs
- **Avoid high-resolution metrics** unless necessary
- **Use log subscription filters** instead of polling

## Troubleshooting

### Missing Metrics

**Causes:**
- Service not publishing yet (wait 1-5 minutes)
- Wrong namespace/dimensions
- Detailed monitoring not enabled (EC2)

**Debug:**

```bash
# List metrics for a namespace
aws cloudwatch list-metrics \
  --namespace AWS/Lambda \
  --dimensions Name=FunctionName,Value=MyFunction
```

### Alarm Stuck in INSUFFICIENT_DATA

**Causes:**
- Metric not being published
- Dimensions mismatch
- Evaluation period too short
- Warm-up period still active (ends early once data fills the window unless `OnlyStartEvaluatingAfterWarmUpPeriodEnds=true`)
- Log alarm: just created/updated (query, schedule, or log group changes reset state), scheduled query role lacks permissions (`EvaluationState` = `EVALUATION_ERROR`, see `StateReason`), or ingestion lag (shift window back with `EndTimeOffset`)

**Debug:**

```bash
# Check if metric has data
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=MyFunction \
  --start-time $(date -d '1 hour ago' -u +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 60 \
  --statistics Sum

# Log alarm: check evaluation state and reason
aws cloudwatch describe-alarms --alarm-types LogAlarm --alarm-names HighErrorCount \
  --query 'LogAlarms[].[StateValue,EvaluationState,StateReason]'
```

### Log Events Not Appearing

**Causes:**
- IAM permissions missing
- CloudWatch Logs agent not running
- Log group doesn't exist

**Debug:**

```bash
# Check log streams
aws logs describe-log-streams \
  --log-group-name /aws/lambda/MyFunction \
  --order-by LastEventTime \
  --descending \
  --limit 5
```

### High CloudWatch Costs

**Check usage:**

```bash
# Get PutLogEvents usage
aws cloudwatch get-metric-statistics \
  --namespace AWS/Logs \
  --metric-name IncomingBytes \
  --dimensions Name=LogGroupName,Value=/aws/lambda/MyFunction \
  --start-time $(date -d '7 days ago' -u +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 86400 \
  --statistics Sum
```

## References

- [CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)
- [CloudWatch Logs User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/)
- [CloudWatch API Reference](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/)
- [CloudWatch CLI Reference](https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/)
- [Logs Insights Query Syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
- [boto3 CloudWatch](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/cloudwatch.html)
