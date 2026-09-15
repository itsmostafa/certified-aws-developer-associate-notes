# Bedrock Model Invocation Patterns

Advanced patterns for invoking foundation models.

## Model-Specific Invocation

### Claude (Anthropic)

```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime')

def invoke_claude(messages, system=None, max_tokens=4096, temperature=1.0):
    body = {
        'anthropic_version': 'bedrock-2023-05-31',
        'max_tokens': max_tokens,
        'temperature': temperature,
        'messages': messages
    }

    if system:
        body['system'] = system
    if temperature != 1.0:
        # Thinking (on by default for Sonnet 5/Opus 5) is incompatible with changed temperature
        body['thinking'] = {'type': 'disabled'}

    response = bedrock.invoke_model(
        modelId='us.anthropic.claude-sonnet-5',
        contentType='application/json',
        accept='application/json',
        body=json.dumps(body)
    )

    return json.loads(response['body'].read())

# Claude Sonnet 5/Opus 5 use adaptive thinking by default: content can include
# 'thinking' blocks before 'text'. Disable with body['thinking'] = {'type': 'disabled'},
# tune with body['output_config'] = {'effort': 'low'|'medium'|'high'}.
# Thinking isn't compatible with changed temperature/top_p/top_k: disable it to set those.
# max_tokens caps thinking + text, so a truncated response may have no text block.
def response_text(result):
    if result['stop_reason'] == 'max_tokens':
        print('Truncated at max_tokens: raise it or lower output_config.effort')
    return next((b['text'] for b in result['content'] if b['type'] == 'text'), '')

# Text generation
result = invoke_claude(
    messages=[{'role': 'user', 'content': 'Explain microservices.'}],
    system='You are a software architect. Be concise.',
    temperature=0.7
)

# With image
import base64

with open('diagram.png', 'rb') as f:
    image_data = base64.standard_b64encode(f.read()).decode()

result = invoke_claude(
    messages=[{
        'role': 'user',
        'content': [
            {
                'type': 'image',
                'source': {
                    'type': 'base64',
                    'media_type': 'image/png',
                    'data': image_data
                }
            },
            {
                'type': 'text',
                'text': 'Describe this architecture diagram.'
            }
        ]
    }]
)
```

### Titan Embeddings (Amazon)

```python
def invoke_titan_embeddings(text, dimensions=1024):
    response = bedrock.invoke_model(
        modelId='amazon.titan-embed-text-v2:0',
        contentType='application/json',
        accept='application/json',
        body=json.dumps({
            'inputText': text,
            'dimensions': dimensions,
            'normalize': True
        })
    )

    result = json.loads(response['body'].read())
    return result['embedding']

# Batch embeddings
def batch_embeddings(texts, dimensions=1024):
    embeddings = []
    for text in texts:
        embedding = invoke_titan_embeddings(text, dimensions)
        embeddings.append(embedding)
    return embeddings
```

### Llama (Meta)

```python
def invoke_llama(prompt, max_tokens=512, temperature=0.7):
    response = bedrock.invoke_model(
        modelId='meta.llama3-70b-instruct-v1:0',
        contentType='application/json',
        accept='application/json',
        body=json.dumps({
            'prompt': prompt,
            'max_gen_len': max_tokens,
            'temperature': temperature,
            'top_p': 0.9
        })
    )

    result = json.loads(response['body'].read())
    return result['generation']

# Format for instruction following
prompt = """<|begin_of_text|><|start_header_id|>system<|end_header_id|>
You are a helpful assistant.<|eot_id|>
<|start_header_id|>user<|end_header_id|>
What is Amazon S3?<|eot_id|>
<|start_header_id|>assistant<|end_header_id|>
"""
```

### Mistral

```python
def invoke_mistral(prompt, max_tokens=512, temperature=0.7):
    response = bedrock.invoke_model(
        modelId='mistral.mistral-large-2402-v1:0',
        contentType='application/json',
        accept='application/json',
        body=json.dumps({
            'prompt': f'<s>[INST] {prompt} [/INST]',
            'max_tokens': max_tokens,
            'temperature': temperature,
            'top_p': 0.9
        })
    )

    result = json.loads(response['body'].read())
    return result['outputs'][0]['text']
```

### Anthropic Messages API (bedrock-runtime)

Use the Anthropic SDK against the `/anthropic` route with a short-term bearer token (`pip install -U anthropic aws-bedrock-token-generator`):

```python
from anthropic import Anthropic
from aws_bedrock_token_generator import provide_token

token = provide_token(region="us-east-1")

client = Anthropic(
    base_url="https://bedrock-runtime.us-east-1.amazonaws.com/anthropic",
    api_key=token,
)

response = client.messages.create(
    model="global.anthropic.claude-sonnet-5",
    max_tokens=4096,  # caps thinking + text
    messages=[{"role": "user", "content": "Explain microservices."}],
)
```

With a Bedrock API key over HTTP: `POST https://bedrock-runtime.{region}.amazonaws.com/anthropic/v1/messages` with headers `x-api-key` and `anthropic-version: 2023-06-01`.

### OpenAI-Compatible APIs

OpenAI SDK code works by changing base URL and key (Bedrock API key):

```bash
export OPENAI_API_KEY="<bedrock-api-key>"
export OPENAI_BASE_URL="https://bedrock-runtime.us-east-1.amazonaws.com/openai/v1"  # recommended
# export OPENAI_BASE_URL="https://bedrock-mantle.us-east-1.api.aws/openai/v1"      # mantle-only features
```

### Web Search (bedrock-mantle, Responses API)

Server-side tool for supported OpenAI GPT models; not available on `bedrock-runtime`. Needs `bedrock-websearch:InvokeSearch` and `bedrock-websearch:InvokeFetch`.

```python
from openai import OpenAI

client = OpenAI()  # OPENAI_BASE_URL=https://bedrock-mantle.us-west-2.api.aws/openai/v1

response = client.responses.create(
    model="openai.gpt-5.6-terra",
    input="Summarize recent guidance on AWS Lambda cold starts.",
    tools=[{
        "type": "web_search",
        "external_web_access": False,  # cache/index only; keeps data in AWS boundary
        "search_context_size": "low",  # low | medium (default) | high
    }],
)
print(response.output_text)
# Citations: output[].content[].annotations[] with type == "url_citation" (must be shown to end users)
```

### Image Generation

Stable Diffusion XL and Titan Text Express are no longer offered. Check [models at a glance](https://docs.aws.amazon.com/bedrock/latest/userguide/model-cards.html) for current image models (e.g. Stability AI Stable Image) and their request bodies.

## Converse API (Unified)

The Converse API provides a unified interface across models.

```python
def converse(messages, model_id, system=None, max_tokens=4096):
    params = {
        'modelId': model_id,
        'messages': messages,
        # No temperature: thinking (default on Sonnet 5/Opus 5) rejects changed values.
        # To set one, also pass additionalModelRequestFields={'thinking': {'type': 'disabled'}}
        'inferenceConfig': {
            'maxTokens': max_tokens
        }
    }

    if system:
        params['system'] = [{'text': system}]

    response = bedrock.converse(**params)
    if response['stopReason'] == 'max_tokens':
        print('Truncated at maxTokens (caps thinking + text)')
    # Skip reasoningContent blocks from thinking models
    return next((c['text'] for c in response['output']['message']['content'] if 'text' in c), '')

# Works with any supported model
result = converse(
    messages=[
        {'role': 'user', 'content': [{'text': 'What is Lambda?'}]}
    ],
    model_id='us.anthropic.claude-sonnet-5',
    system='Be concise.'
)
```

### Converse with Tool Use

```python
def converse_with_tools(messages, tools, model_id):
    response = bedrock.converse(
        modelId=model_id,
        messages=messages,
        toolConfig={
            'tools': tools
        }
    )

    output = response['output']['message']

    # Check if model wants to use a tool
    if response['stopReason'] == 'tool_use':
        tool_use = next(
            block for block in output['content']
            if 'toolUse' in block
        )
        return {
            'tool_name': tool_use['toolUse']['name'],
            'tool_input': tool_use['toolUse']['input'],
            'tool_use_id': tool_use['toolUse']['toolUseId']
        }

    return {'text': next((c['text'] for c in output['content'] if 'text' in c), '')}

# Define tools
tools = [{
    'toolSpec': {
        'name': 'get_weather',
        'description': 'Get current weather for a location',
        'inputSchema': {
            'json': {
                'type': 'object',
                'properties': {
                    'location': {
                        'type': 'string',
                        'description': 'City name'
                    }
                },
                'required': ['location']
            }
        }
    }
}]

# Invoke
result = converse_with_tools(
    messages=[
        {'role': 'user', 'content': [{'text': 'What is the weather in Seattle?'}]}
    ],
    tools=tools,
    model_id='us.anthropic.claude-sonnet-5'
)
```

## RAG with Knowledge Bases

```python
bedrock_agent = boto3.client('bedrock-agent-runtime')

def rag_query(query, knowledge_base_id, model_arn):
    response = bedrock_agent.retrieve_and_generate(
        input={'text': query},
        retrieveAndGenerateConfiguration={
            'type': 'KNOWLEDGE_BASE',
            'knowledgeBaseConfiguration': {
                'knowledgeBaseId': knowledge_base_id,
                'modelArn': model_arn,
                'retrievalConfiguration': {
                    'vectorSearchConfiguration': {
                        'numberOfResults': 5
                    }
                }
            }
        }
    )

    return {
        'answer': response['output']['text'],
        'citations': response.get('citations', [])
    }

# Usage
result = rag_query(
    query='How do I configure S3 bucket policies?',
    knowledge_base_id='KNOWLEDGE_BASE_ID',
    model_arn='arn:aws:bedrock:us-east-1:123456789012:inference-profile/us.anthropic.claude-sonnet-5'
)
```

### Retrieve Only (No Generation)

```python
def retrieve_context(query, knowledge_base_id, num_results=5):
    response = bedrock_agent.retrieve(
        knowledgeBaseId=knowledge_base_id,
        retrievalQuery={'text': query},
        retrievalConfiguration={
            'vectorSearchConfiguration': {
                'numberOfResults': num_results
            }
        }
    )

    return [
        {
            'text': result['content']['text'],
            'score': result['score'],
            'source': result['location']
        }
        for result in response['retrievalResults']
    ]
```

### Agentic Retrieval (Managed Knowledge Bases)

SDK-only event stream. Needs `bedrock:AgenticRetrieveStream`, `bedrock:Retrieve`, `bedrock:GetDocumentContent`, `bedrock:InvokeModelWithResponseStream`.

```python
def agentic_retrieve(query, knowledge_base_id):
    response = bedrock_agent.agentic_retrieve_stream(
        messages=[{'role': 'user', 'content': {'text': query}}],
        retrievers=[{
            'configuration': {'knowledgeBase': {'knowledgeBaseId': knowledge_base_id}}
        }],  # up to 5
        agenticRetrieveConfiguration={
            'foundationModelType': 'MANAGED',   # or CUSTOM + foundationModelConfiguration
            'rerankingModelType': 'MANAGED'
        },
        generateResponse=True  # default
    )

    for event in response['stream']:
        if 'responseEvent' in event:
            print(event['responseEvent']['text'], end='')
        elif 'result' in event:
            # results[] deduplicated across iterations; generatedResponse.answer/citations
            return event['result']
```

- AgentCore Memory: `memoryConfiguration={'memoryId': ..., 'sessionBinding': {'actorId': ..., 'sessionId': ...}, 'retrievalConfigs': [{'namespace': ...}]}` (needs `sessionBinding`, `retrievalConfigs`, or both). With `sessionBinding`, `messages` must hold only the current user query; `persistenceMode` `DEFAULT` writes the exchange back (requires `generateResponse=True`), `NONE` reads only
- Guardrails via `policyConfiguration` support only `BLOCK` (no `MASK`)

### Native Multimodal Knowledge Bases

- Choose `twelvelabs.marengo-embed-3-0-v1:0` as the embedding model when creating a managed KB; parsing strategy must be `MULTI_MODAL_EMBEDDINGS` (no text chunking; configure audio/video segmentation instead)
- Requires a multimodal storage S3 destination separate from the data source bucket; add a lifecycle rule only on `aws/bedrock/knowledge_bases/<kb-id>/<ds-id>/transient_data`
- Query with text through `retrieve`; results carry segment start/end times. `retrieve_and_generate` and image queries are not supported

## Guardrails

```python
def invoke_with_guardrails(prompt, guardrail_id, guardrail_version):
    response = bedrock.invoke_model(
        modelId='us.anthropic.claude-sonnet-5',
        contentType='application/json',
        accept='application/json',
        body=json.dumps({
            'anthropic_version': 'bedrock-2023-05-31',
            'max_tokens': 4096,
            'messages': [{'role': 'user', 'content': prompt}]
        }),
        guardrailIdentifier=guardrail_id,
        guardrailVersion=guardrail_version
    )

    result = json.loads(response['body'].read())

    # Check if guardrail intervened
    if 'amazon-bedrock-guardrailAction' in response['ResponseMetadata']['HTTPHeaders']:
        return {
            'blocked': True,
            'reason': 'Content policy violation'
        }

    return {
        'blocked': False,
        'text': next((b['text'] for b in result['content'] if b['type'] == 'text'), '')
    }
```

## Batch Inference

```python
import boto3

bedrock = boto3.client('bedrock')

def create_batch_job(input_s3_uri, output_s3_uri, model_id, role_arn,
                     invocation_type='InvokeModel'):
    response = bedrock.create_model_invocation_job(
        jobName=f'batch-job-{int(time.time())}',
        modelId=model_id,
        roleArn=role_arn,
        modelInvocationType=invocation_type,  # 'InvokeModel' (default) or 'Converse'
        inputDataConfig={
            's3InputDataConfig': {
                's3Uri': input_s3_uri
            }
        },
        outputDataConfig={
            's3OutputDataConfig': {
                's3Uri': output_s3_uri
            }
        }
    )

    return response['jobArn']

# Input format (JSONL file in S3)
# InvokeModel: modelInput is the model-specific body
# {"recordId": "1", "modelInput": {"anthropic_version": "...", "messages": [...]}}
# Converse: modelInput is a Converse request body
# {"recordId": "2", "modelInput": {"messages": [{"role": "user", "content": [{"text": "..."}]}], "inferenceConfig": {"maxTokens": 1024}}}
# Output record order is not guaranteed
```

```bash
aws bedrock create-model-invocation-job \
  --job-name my-batch-job \
  --role-arn arn:aws:iam::123456789012:role/BedrockBatchRole \
  --model-id <model-or-inference-profile-id> \
  --model-invocation-type Converse \
  --input-data-config '{"s3InputDataConfig": {"s3Uri": "s3://my-bucket/input/"}}' \
  --output-data-config '{"s3OutputDataConfig": {"s3Uri": "s3://my-bucket/output/"}}'
```

## Error Handling

```python
from botocore.exceptions import ClientError
import time

class BedrockInvoker:
    def __init__(self, model_id):
        self.bedrock = boto3.client('bedrock-runtime')
        self.model_id = model_id

    def invoke(self, body, max_retries=3):
        last_error = None

        for attempt in range(max_retries):
            try:
                response = self.bedrock.invoke_model(
                    modelId=self.model_id,
                    contentType='application/json',
                    accept='application/json',
                    body=json.dumps(body)
                )
                return json.loads(response['body'].read())

            except ClientError as e:
                error_code = e.response['Error']['Code']
                last_error = e

                if error_code == 'ThrottlingException':
                    wait_time = (2 ** attempt) + random.random()
                    time.sleep(wait_time)
                elif error_code == 'ModelNotReadyException':
                    time.sleep(5)
                elif error_code == 'ValidationException':
                    raise  # Don't retry validation errors
                else:
                    raise

        raise last_error
```

## Provisioned Throughput

Not available for inference profiles or Legacy models.

```bash
# Create provisioned throughput
aws bedrock create-provisioned-model-throughput \
  --model-id <base-model-id-or-custom-model-arn> \
  --provisioned-model-name my-claude-capacity \
  --model-units 1

# Use provisioned model
aws bedrock-runtime invoke-model \
  --model-id arn:aws:bedrock:us-east-1:123456789012:provisioned-model/my-claude-capacity \
  --body '...' \
  response.json
```

```python
# Invoke provisioned model
response = bedrock.invoke_model(
    modelId='arn:aws:bedrock:us-east-1:123456789012:provisioned-model/my-claude-capacity',
    contentType='application/json',
    accept='application/json',
    body=json.dumps(body)
)
```

## VPC Endpoint

```yaml
# CloudFormation for private Bedrock access
Resources:
  BedrockEndpoint:
    Type: AWS::EC2::VPCEndpoint
    Properties:
      VpcId: !Ref VPC
      ServiceName: !Sub com.amazonaws.${AWS::Region}.bedrock-runtime
      VpcEndpointType: Interface
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2
      SecurityGroupIds:
        - !Ref BedrockSecurityGroup
      PrivateDnsEnabled: true

  BedrockSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          SourceSecurityGroupId: !Ref AppSecurityGroup
```
