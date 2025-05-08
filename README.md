
Updated Project Structure
markdown
Copy code
eventbridge-lambda-dynamo/
├── app.py
├── cdk.json
├── requirements.txt
├── eventbridge_lambda_dynamo/
│   ├── __init__.py
│   ├── eventbridge_stack.py
│   ├── lambda_stack.py
│   └── dynamodb_stack.py
├── .github/
│   └── workflows/
│       └── deploy.yml
✅ 1. app.py
python
Copy code
#!/usr/bin/env python3
import aws_cdk as cdk
from eventbridge_lambda_dynamo.eventbridge_stack import EventBridgeStack
from eventbridge_lambda_dynamo.lambda_stack import LambdaStack
from eventbridge_lambda_dynamo.dynamodb_stack import DynamoDBStack

app = cdk.App()

event_stack = EventBridgeStack(app, "EventBridgeStack")
lambda_stack = LambdaStack(app, "LambdaStack")
lambda_stack.add_dependency(event_stack)

dynamodb_stack = DynamoDBStack(app, "DynamoDBStack")
dynamodb_stack.add_dependency(lambda_stack)

app.synth()
✅ 2. eventbridge_stack.py
python
Copy code
from aws_cdk import Stack, aws_events as events
from constructs import Construct

class EventBridgeStack(Stack):
    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        events.Rule(
            self, "SampleEventRule",
            event_pattern=events.EventPattern(
                source=["custom.source"]
            )
        )
✅ 3. lambda_stack.py
python
Copy code
from aws_cdk import Stack, aws_lambda as _lambda
from constructs import Construct

class LambdaStack(Stack):
    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        _lambda.Function(
            self, "SampleLambda",
            runtime=_lambda.Runtime.PYTHON_3_9,
            handler="index.handler",
            code=_lambda.Code.from_inline(
                "def handler(event, context):\n    print('Hello from Lambda')"
            )
        )
✅ 4. dynamodb_stack.py
python
Copy code
from aws_cdk import Stack, aws_dynamodb as ddb
from constructs import Construct

class DynamoDBStack(Stack):
    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        ddb.Table(
            self, "SampleTable",
            partition_key={"name": "id", "type": ddb.AttributeType.STRING}
        )
✅ 5. .github/workflows/deploy.yml
Same as before — it’ll automatically synthesize and deploy stacks in order due to .add_dependency().

yaml
Copy code
name: Deploy CDK Stacks

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source
        uses: actions/checkout@v3

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Install AWS CDK
        run: npm install -g aws-cdk

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy CDK
        run: cdk deploy --require-approval never
Would you like me to generate a downloadable .zip of this full working project?







