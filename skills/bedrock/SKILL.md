---
name: bedrock
description: AWS Bedrock foundation models for generative AI. Use when invoking foundation models, building AI applications, creating embeddings, configuring model access, or implementing RAG patterns.
last_updated: "2026-09-14"
doc_source: https://docs.aws.amazon.com/bedrock/latest/userguide/
---

# AWS Bedrock

Amazon Bedrock provides access to foundation models (FMs) from AI companies through a unified API. Build generative AI applications with text generation, embeddings, and image generation capabilities.

## Table of Contents

- [Core Concepts](#core-concepts)
- [Common Patterns](#common-patterns)
- [CLI Reference](#cli-reference)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [References](#references)

## Core Concepts

### Foundation Models

Pre-trained models available through Bedrock:
- **Claude** (Anthropic): Text generation, analysis, coding
- **Nova / Titan** (Amazon): Text, multimodal, embeddings
- **GPT / gpt-oss** (OpenAI): Text generation, reasoning
- **Llama** (Meta): Open-weight text generation
- **Mistral**: Efficient text generation
- **Stable Image** (Stability AI): Image generation and editing

### Model Access

In commercial Regions, access to all serverless models is enabled by default (no console opt-in). In GovCloud (US), models are still enabled manually on the Model access page (third-party models also in the linked commercial account):
- First invocation of a third-party model auto-subscribes via AWS Marketplace (up to 15 min); caller needs `aws-marketplace:Subscribe`, `Unsubscribe`, `ViewSubscriptions`
- Anthropic models on `bedrock-runtime` need a one-time use case form per account/org (`put-use-case-for-model-access`)
- Invoking implies EULA acceptance; to block a model, deny both `bedrock:InvokeModel` and `bedrock:InvokeModelWithResponseStream` on it (SCP/IAM); streaming APIs such as `ConverseStream` use the latter. Denying `aws-marketplace:Subscribe` alone does not block first use

### Endpoints

| Endpoint | APIs | Use for |
|----------|------|---------|
| `bedrock-runtime.{region}.amazonaws.com` (recommended) | InvokeModel, Converse, Anthropic Messages (`/anthropic`), OpenAI Responses/Chat Completions (`/openai/v1`) | Guardrails, cross-Region inference, prompt routing, application inference profiles |
| `bedrock-mantle.{region}.api.aws` | OpenAI Responses/Chat Completions (`/openai/v1`), Anthropic Messages | Server-side tools (Web Search), `background=true` async, Projects/Workspaces, single-Region access to CRIS-only models |

- Same per-token price on both; auth via SigV4 or Bedrock API key (`AWS_BEARER_TOKEN_BEDROCK`)
- IAM: `bedrock:InvokeModel` (runtime) vs `bedrock-mantle:CreateInference` (mantle)
- Responses API on `bedrock-runtime` is synchronous only and has no server-side tools

### Inference Profiles and Model Lifecycle

- Newer models (e.g. Claude Sonnet 5) have no in-Region on-demand ID on `bedrock-runtime`: use a geo (`us.`, `eu.`, `au.`) or `global.` inference profile ID as `modelId`
- Lifecycle is `Active` -> `Legacy` -> `EOL` (see `modelLifecycle` in `get-foundation-model`). Legacy: no new Provisioned Throughput, fine-tuning, or quota increases; EOL: requests fail
- Model cards list an "EOL no sooner than" date; check before pinning a model ID

### Knowledge Bases and Agents

- **Managed knowledge bases** (`type: MANAGED`): Bedrock runs storage, indexing, and retrieval. Only type that supports `AgenticRetrieveStream` (query decomposition, iterative retrieval, optional AgentCore Memory via `memoryConfiguration`)
- **Native multimodal** managed KBs embed video/audio/image directly with TwelveLabs Marengo Embed 3.0 (`twelvelabs.marengo-embed-3-0-v1:0`); query with text via `Retrieve` only (no `RetrieveAndGenerate`)
- **Bedrock Agents Classic** is in maintenance mode: closed to new accounts since July 30, 2026 (`CreateAgent`/`InvokeInlineAgent` return 403 without prior 12-month usage), model catalog frozen. Build new agents on Amazon Bedrock AgentCore

### Inference Types

| Type | Use Case | Pricing |
|------|----------|---------|
| **On-Demand** | Variable workloads | Per token |
| **Provisioned Throughput** | Consistent high-volume | Hourly commitment |
| **Batch Inference** | Async large-scale | Discounted per token |

## Common Patterns

### Invoke Model (Text Generation)

**AWS CLI:**

```bash
# Invoke Claude
aws bedrock-runtime invoke-model \
  --model-id us.anthropic.claude-sonnet-5 \
  --content-type application/json \
  --accept application/json \
  --cli-binary-format raw-in-base64-out \
  --body '{
    "anthropic_version": "bedrock-2023-05-31",
    "max_tokens": 4096,
    "messages": [
      {"role": "user", "content": "Explain AWS Lambda in 3 sentences."}
    ]
  }' \
  response.json

# Claude Sonnet 5/Opus 5 think by default: content may start with a thinking block
cat response.json | jq -r '.content[] | select(.type=="text") | .text'
```

**boto3:**

```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime')

def invoke_claude(prompt, max_tokens=4096):
    response = bedrock.invoke_model(
        modelId='us.anthropic.claude-sonnet-5',
        contentType='application/json',
        accept='application/json',
        body=json.dumps({
            'anthropic_version': 'bedrock-2023-05-31',
            'max_tokens': max_tokens,
            'messages': [
                {'role': 'user', 'content': prompt}
            ]
        })
    )

    result = json.loads(response['body'].read())
    # Skip thinking blocks (adaptive thinking is on by default for Sonnet 5).
    # max_tokens caps thinking + text, so a truncated response may have no text block.
    if result['stop_reason'] == 'max_tokens':
        print('Truncated at max_tokens: raise it or lower output_config.effort')
    return next((b['text'] for b in result['content'] if b['type'] == 'text'), '')

# Usage
response = invoke_claude('What is Amazon S3?')
print(response)
```

### Streaming Response

```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime')

def stream_claude(prompt):
    response = bedrock.invoke_model_with_response_stream(
        modelId='us.anthropic.claude-sonnet-5',
        contentType='application/json',
        accept='application/json',
        body=json.dumps({
            'anthropic_version': 'bedrock-2023-05-31',
            'max_tokens': 4096,
            'messages': [
                {'role': 'user', 'content': prompt}
            ]
        })
    )

    for event in response['body']:
        chunk = json.loads(event['chunk']['bytes'])
        if chunk['type'] == 'content_block_delta':
            yield chunk['delta'].get('text', '')

# Usage
for text in stream_claude('Write a haiku about cloud computing.'):
    print(text, end='', flush=True)
```

### Generate Embeddings

```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime')

def get_embedding(text):
    response = bedrock.invoke_model(
        modelId='amazon.titan-embed-text-v2:0',
        contentType='application/json',
        accept='application/json',
        body=json.dumps({
            'inputText': text,
            'dimensions': 1024,
            'normalize': True
        })
    )

    result = json.loads(response['body'].read())
    return result['embedding']

# Usage
embedding = get_embedding('AWS Lambda is a serverless compute service.')
print(f'Embedding dimension: {len(embedding)}')
```

### Conversation with History

```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime')

class Conversation:
    def __init__(self, system_prompt=None):
        self.messages = []
        self.system = system_prompt

    def chat(self, user_message):
        self.messages.append({
            'role': 'user',
            'content': user_message
        })

        body = {
            'anthropic_version': 'bedrock-2023-05-31',
            'max_tokens': 4096,
            'messages': self.messages
        }

        if self.system:
            body['system'] = self.system

        response = bedrock.invoke_model(
            modelId='us.anthropic.claude-sonnet-5',
            contentType='application/json',
            accept='application/json',
            body=json.dumps(body)
        )

        result = json.loads(response['body'].read())
        if result['stop_reason'] == 'max_tokens':
            # max_tokens caps thinking + text; don't store a truncated/empty turn
            self.messages.pop()
            raise RuntimeError('Truncated at max_tokens: raise it or lower output_config.effort')
        assistant_message = next(
            (b['text'] for b in result['content'] if b['type'] == 'text'), ''
        )

        self.messages.append({
            'role': 'assistant',
            'content': assistant_message
        })

        return assistant_message

# Usage
conv = Conversation(system_prompt='You are an AWS solutions architect.')
print(conv.chat('What database should I use for a chat application?'))
print(conv.chat('What about for time-series data?'))
```

### List Available Models

```bash
# List all foundation models
aws bedrock list-foundation-models \
  --query 'modelSummaries[*].[modelId,modelName,providerName]' \
  --output table

# Filter by provider
aws bedrock list-foundation-models \
  --by-provider anthropic \
  --query 'modelSummaries[*].modelId'

# Get model details (includes modelLifecycle.status)
aws bedrock get-foundation-model \
  --model-identifier anthropic.claude-sonnet-5
```

### Check Model Access

```bash
# agreementAvailability.status AVAILABLE / NOT_AVAILABLE, authorizationStatus
aws bedrock get-foundation-model-availability \
  --model-id anthropic.claude-sonnet-5

# Anthropic one-time use case form (base64-encoded JSON:
# companyName, companyWebsite, intendedUsers, industryOption, otherIndustryOption, useCases)
aws bedrock put-use-case-for-model-access --form-data <base64-json>

# Programmatic agreement for third-party models
aws bedrock list-foundation-model-agreement-offers --model-id <model-id>
aws bedrock create-foundation-model-agreement --model-id <model-id> --offer-token <token>
```

### Count Tokens

```bash
# Free; returns inputTokens. Not supported for every model (e.g. CRIS-only Claude models)
aws bedrock-runtime count-tokens \
  --model-id anthropic.claude-3-5-haiku-20241022-v1:0 \
  --input '{"converse": {"messages": [{"role": "user", "content": [{"text": "Hello"}]}]}}'
```

## CLI Reference

### Bedrock (Control Plane)

| Command | Description |
|---------|-------------|
| `aws bedrock list-foundation-models` | List available models |
| `aws bedrock get-foundation-model` | Get model details |
| `aws bedrock list-custom-models` | List fine-tuned models |
| `aws bedrock create-model-customization-job` | Start fine-tuning |
| `aws bedrock list-provisioned-model-throughputs` | List provisioned capacity |
| `aws bedrock get-foundation-model-availability` | Check access/agreement status for a model |
| `aws bedrock put-use-case-for-model-access` | Submit Anthropic first-time use case form |
| `aws bedrock list-inference-profiles` | List system/application inference profiles |
| `aws bedrock create-model-invocation-job` | Start batch job (`--model-invocation-type InvokeModel\|Converse`) |

### Bedrock Runtime (Data Plane)

| Command | Description |
|---------|-------------|
| `aws bedrock-runtime invoke-model` | Invoke model synchronously |
| `aws bedrock-runtime converse` | Multi-turn conversation API |
| `aws bedrock-runtime count-tokens` | Count input tokens (`--input` with `invokeModel` or `converse`) |
| `aws bedrock-runtime apply-guardrail` | Evaluate content against a guardrail |

`InvokeModelWithResponseStream` and `ConverseStream` are SDK-only (not in AWS CLI v2).

### Bedrock Agent Runtime

| Command | Description |
|---------|-------------|
| `aws bedrock-agent-runtime retrieve` | Query knowledge base |
| `aws bedrock-agent-runtime retrieve-and-generate` | RAG query |

`InvokeAgent`, `RetrieveAndGenerateStream`, and `AgenticRetrieveStream` are SDK-only (event streams).

## Best Practices

### Cost Optimization

- **Use appropriate models**: Smaller models for simple tasks
- **Set max_tokens**: Limit output length when possible
- **Cache responses**: For repeated identical queries
- **Batch when possible**: Use batch inference for bulk processing
- **Monitor usage**: Set up CloudWatch alarms for cost
- **Global inference profiles**: ~10% cheaper than geo profiles when data residency allows
- **Thinking tokens bill as output**: Claude Sonnet 5/Opus 5 think by default; pass `"thinking": {"type": "disabled"}` or lower `output_config.effort` if not needed, and revisit `max_tokens` (it caps thinking + text)
- **Converse batch format**: `--model-invocation-type Converse` keeps one request shape across models
- **Cost attribution**: Tag IAM principals as cost allocation tags (works on both endpoints)

### Performance

- **Use streaming**: For better user experience with long outputs
- **Connection pooling**: Reuse boto3 clients
- **Regional deployment**: Use closest region to reduce latency
- **Provisioned throughput**: For consistent high-volume workloads
- **Endpoint choice**: Default to `bedrock-runtime`; use `bedrock-mantle` only for mantle-only features

### Security

- **Least privilege IAM**: Only grant needed model access
- **VPC endpoints**: Keep traffic private
- **Guardrails**: Implement content filtering
- **Audit with CloudTrail**: Track model invocations
- **Web Search (mantle)**: Set `external_web_access: false` to keep Fetch inside the AWS boundary; `AmazonBedrockFullAccess` lacks `bedrock-websearch:ExternalWebAccess`, so the default `true` silently fails Fetch
- **Pin active models**: Check `modelLifecycle` and migrate off `Legacy` models before EOL

### IAM Permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": [
        "arn:aws:bedrock:us-east-1:123456789012:inference-profile/us.anthropic.claude-sonnet-5",
        "arn:aws:bedrock:us-east-1::foundation-model/amazon.titan-embed-text-v2:0"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "bedrock:InvokeModel",
        "bedrock:InvokeModelWithResponseStream"
      ],
      "Resource": "arn:aws:bedrock:*::foundation-model/anthropic.claude-sonnet-5",
      "Condition": {
        "StringEquals": {
          "bedrock:InferenceProfileArn": "arn:aws:bedrock:us-east-1:123456789012:inference-profile/us.anthropic.claude-sonnet-5"
        }
      }
    }
  ]
}
```

Inference profiles need access to the profile ARN plus the foundation model in every destination Region (list them with `aws bedrock get-inference-profile --inference-profile-identifier <id>`, `models` field). SCPs that deny Regions must allow those destinations (or exempt via `bedrock:InferenceProfileArn`).

## Troubleshooting

### AccessDeniedException

**Causes:**
- Missing `aws-marketplace:Subscribe` on first use of a third-party model (auto-subscription fails; may take ~2 min after fixing)
- Anthropic use case form not submitted
- IAM policy missing `bedrock:InvokeModel`, or missing destination-Region foundation-model ARNs for an inference profile
- Wrong model ID or region
- Bedrock Agents Classic: "Bedrock Agents is in Maintenance Mode" 403 on `CreateAgent`/`InvokeInlineAgent` in accounts without prior usage (use AgentCore)

**Debug:**

```bash
# Check model access status
aws bedrock get-foundation-model-availability \
  --model-id anthropic.claude-sonnet-5

# Test IAM permissions
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:role/my-role \
  --action-names bedrock:InvokeModel \
  --resource-arns "arn:aws:bedrock:us-east-1:123456789012:inference-profile/us.anthropic.claude-sonnet-5"

# The profile ARN can pass while cross-Region routing is still denied: also simulate each
# destination-Region model ARN (models field of get-inference-profile) with the profile context
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:role/my-role \
  --action-names bedrock:InvokeModel \
  --resource-arns "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-sonnet-5" \
                  "arn:aws:bedrock:us-west-2::foundation-model/anthropic.claude-sonnet-5" \
  --context-entries '[{"ContextKeyName":"bedrock:InferenceProfileArn","ContextKeyType":"string","ContextKeyValues":["arn:aws:bedrock:us-east-1:123456789012:inference-profile/us.anthropic.claude-sonnet-5"]}]'
```

### ModelNotReadyException

**Cause:** Model is still being provisioned or temporarily unavailable.

**Solution:** Implement retry with exponential backoff:

```python
import time
from botocore.exceptions import ClientError

def invoke_with_retry(bedrock, body, max_retries=3):
    for attempt in range(max_retries):
        try:
            return bedrock.invoke_model(
                modelId='us.anthropic.claude-sonnet-5',
                body=json.dumps(body)
            )
        except ClientError as e:
            if e.response['Error']['Code'] == 'ModelNotReadyException':
                time.sleep(2 ** attempt)
            else:
                raise
    raise Exception('Max retries exceeded')
```

### ThrottlingException

**Causes:**
- Exceeded per-model tokens-per-minute (input + output combined on `bedrock-runtime`) or tokens-per-day quota
- RPM quota (model-specific; some models have none)
- Too many concurrent requests

**Solutions:**
- Request quota increase (request "Cross-Region InvokeModel tokens per minute for <model>" to cover TPM/TPD together; not granted for Legacy models)
- Lower `max_tokens`: it affects quota deduction
- Use a cross-Region inference profile for higher throughput
- Implement exponential backoff
- Consider provisioned throughput

### ValidationException

**Common issues:**
- Invalid model ID, or model is EOL
- Error mentions on-demand throughput not supported for the model ID: use an inference profile ID (`us.`/`global.` prefix)
- Malformed request body
- max_tokens exceeds model limit
- `thinking.type: "enabled"` with `budget_tokens` on models that only accept `adaptive`/`disabled` (e.g. Claude Sonnet 5)
- `output_config.format` (structured outputs) sent to `bedrock-mantle` (use Converse/InvokeModel on `bedrock-runtime`)

**Debug:**

```python
# Check model-specific requirements
aws bedrock get-foundation-model \
  --model-identifier anthropic.claude-sonnet-5 \
  --query 'modelDetails.[inferenceTypesSupported,modelLifecycle.status]'
```

## References

- [Bedrock User Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/)
- [Bedrock API Reference](https://docs.aws.amazon.com/bedrock/latest/APIReference/)
- [Bedrock Runtime API](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_Operations_Amazon_Bedrock_Runtime.html)
- [Model Parameters](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters.html)
- [Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/)
- [Endpoints](https://docs.aws.amazon.com/bedrock/latest/userguide/endpoints.html)
- [Models at a Glance](https://docs.aws.amazon.com/bedrock/latest/userguide/model-cards.html)
- [Model Lifecycle](https://docs.aws.amazon.com/bedrock/latest/userguide/model-lifecycle.html)
- [Agents Classic Maintenance Mode](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-classic-maintenance-mode.html)
