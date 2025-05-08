
Here's a clean, production-ready AWS CDK setup (Python) for sequential deployment with connections between Lambda, DynamoDB, and EventBridge.

✅ Architecture Overview
📦 Stacks (deployed in order):

DynamoDBStack → creates the table

LambdaStack → creates a Lambda and connects it to DynamoDB

EventBridgeStack → creates a rule to invoke Lambda on events

All stacks are deployed sequentially using .add_dependency().

📁 Project Structure
markdown
Copy code
cdk-app/
├── app.py
├── requirements.txt
├── cdk.json
├── eventbridge_lambda_dynamo/
│   ├── __init__.py
│   ├── dynamodb_stack.py
│   ├── lambda_stack.py
│   └── eventbridge_stack.py
✅ app.py
python
Copy code
#!/usr/bin/env python3
import aws_cdk as cdk
from eventbridge_lambda_dynamo.dynamodb_stack import DynamoDBStack
from eventbridge_lambda_dynamo.lambda_stack import LambdaStack
from eventbridge_lambda_dynamo.eventbridge_stack import EventBridgeStack

app = cdk.App()

# Create DynamoDB first
dynamo_stack = DynamoDBStack(app, "DynamoDBStack")

# Then Lambda (uses DynamoDB table)
lambda_stack = LambdaStack(app, "LambdaStack", table=dynamo_stack.table)
lambda_stack.add_dependency(dynamo_stack)

# Finally EventBridge (triggers Lambda)
eventbridge_stack = EventBridgeStack(app, "EventBridgeStack", lambda_fn=lambda_stack.lambda_fn)
eventbridge_stack.add_dependency(lambda_stack)

app.synth()
✅ dynamodb_stack.py
python
Copy code
from aws_cdk import aws_dynamodb as ddb, Stack
from constructs import Construct

class DynamoDBStack(Stack):
    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        self.table = ddb.Table(
            self, "MyTable",
            partition_key={"name": "id", "type": ddb.AttributeType.STRING},
            billing_mode=ddb.BillingMode.PAY_PER_REQUEST
        )
✅ lambda_stack.py
python
Copy code
from aws_cdk import (
    Stack,
    aws_lambda as _lambda,
    aws_dynamodb as ddb,
)
from constructs import Construct

class LambdaStack(Stack):
    def __init__(self, scope: Construct, id: str, table: ddb.ITable, **kwargs):
        super().__init__(scope, id, **kwargs)

        self.lambda_fn = _lambda.Function(
            self, "MyFunction",
            runtime=_lambda.Runtime.PYTHON_3_9,
            handler="index.handler",
            code=_lambda.Code.from_inline("""
def handler(event, context):
    print("Event received:", event)
""")
        )

        table.grant_read_write_data(self.lambda_fn)
✅ eventbridge_stack.py
python
Copy code
from aws_cdk import (
    Stack,
    aws_events as events,
    aws_events_targets as targets,
    aws_lambda as _lambda
)
from constructs import Construct

class EventBridgeStack(Stack):
    def __init__(self, scope: Construct, id: str, lambda_fn: _lambda.IFunction, **kwargs):
        super().__init__(scope, id, **kwargs)

        rule = events.Rule(
            self, "SampleEventRule",
            event_pattern=events.EventPattern(
                source=["my.custom.source"]
            )
        )

        rule.add_target(targets.LambdaFunction(lambda_fn))
✅ requirements.txt
txt
Copy code
aws-cdk-lib==2.117.0
constructs>=10.0.0
✅ GitHub Actions Tip
Deploy in .github/workflows/deploy.yml using:

yaml
Copy code
- run: cdk deploy --all --require-approval never
Would you like me to export this as a downloadable ZIP project?








