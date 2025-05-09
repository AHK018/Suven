
# Directory: eventbridge_crud_cdk/

# ==============================
# File: app.py (Entry point)
# ==============================
from aws_cdk import App
from stacks.dynamodb_stack import DynamoDBStack
from stacks.eventbridge_stack import EventBridgeStack
from stacks.lambda_create_stack import LambdaCreateStack
from stacks.lambda_read_stack import LambdaReadStack
from stacks.lambda_update_stack import LambdaUpdateStack
from stacks.lambda_delete_stack import LambdaDeleteStack

app = App()

dynamo_stack = DynamoDBStack(app, "DynamoDBStack")
eventbridge_stack = EventBridgeStack(app, "EventBridgeStack")

LambdaCreateStack(app, "LambdaCreateStack",
    table=dynamo_stack.table,
    bus=eventbridge_stack.bus,
    detail_type="CREATE_ITEM")

LambdaReadStack(app, "LambdaReadStack",
    table=dynamo_stack.table,
    bus=eventbridge_stack.bus,
    detail_type="READ_ITEM")

LambdaUpdateStack(app, "LambdaUpdateStack",
    table=dynamo_stack.table,
    bus=eventbridge_stack.bus,
    detail_type="UPDATE_ITEM")

LambdaDeleteStack(app, "LambdaDeleteStack",
    table=dynamo_stack.table,
    bus=eventbridge_stack.bus,
    detail_type="DELETE_ITEM")

app.synth()


# =================================
# File: stacks/dynamodb_stack.py
# =================================
from aws_cdk import Stack
from aws_cdk.aws_dynamodb import Table, AttributeType
from constructs import Construct

class DynamoDBStack(Stack):
    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        self.table = Table(self, "ItemsTable",
            partition_key={"name": "id", "type": AttributeType.STRING},
            table_name="ItemsTable"
        )


# ======================================
# File: stacks/eventbridge_stack.py
# ======================================
from aws_cdk import Stack
from aws_cdk.aws_events import EventBus
from constructs import Construct

class EventBridgeStack(Stack):
    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)
        self.bus = EventBus(self, "CrudEventBus", event_bus_name="CrudEventBus")


# ================================================
# File: stacks/lambda_create_stack.py
# ================================================
from aws_cdk import Stack, Duration
from aws_cdk.aws_lambda import Function, Runtime, Code
from aws_cdk.aws_events_targets import LambdaFunction
from aws_cdk.aws_events import Rule, EventPattern
from aws_cdk.aws_dynamodb import ITable
from aws_cdk.aws_events import IEventBus
from constructs import Construct

class LambdaCreateStack(Stack):
    def __init__(self, scope: Construct, id: str, table: ITable, bus: IEventBus, detail_type: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        lambda_fn = Function(self, "CreateLambda",
            runtime=Runtime.PYTHON_3_9,
            handler="index.handler",
            code=Code.from_asset("lambdas/create"),
            environment={"TABLE_NAME": table.table_name},
            timeout=Duration.seconds(10)
        )

        table.grant_write_data(lambda_fn)

        Rule(self, "CreateRule",
            event_bus=bus,
            event_pattern=EventPattern(
                source=["crud.app"],
                detail_type=[detail_type]
            ),
            targets=[LambdaFunction(lambda_fn)]
        )


# ================================================
# File: stacks/lambda_read_stack.py
# ================================================
from aws_cdk import Stack, Duration
from aws_cdk.aws_lambda import Function, Runtime, Code
from aws_cdk.aws_events_targets import LambdaFunction
from aws_cdk.aws_events import Rule, EventPattern
from aws_cdk.aws_dynamodb import ITable
from aws_cdk.aws_events import IEventBus
from constructs import Construct

class LambdaReadStack(Stack):
    def __init__(self, scope: Construct, id: str, table: ITable, bus: IEventBus, detail_type: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        lambda_fn = Function(self, "ReadLambda",
            runtime=Runtime.PYTHON_3_9,
            handler="index.handler",
            code=Code.from_asset("lambdas/read"),
            environment={"TABLE_NAME": table.table_name},
            timeout=Duration.seconds(10)
        )

        table.grant_read_data(lambda_fn)

        Rule(self, "ReadRule",
            event_bus=bus,
            event_pattern=EventPattern(
                source=["crud.app"],
                detail_type=[detail_type]
            ),
            targets=[LambdaFunction(lambda_fn)]
        )


# ================================================
# File: stacks/lambda_update_stack.py
# ================================================
from aws_cdk import Stack, Duration
from aws_cdk.aws_lambda import Function, Runtime, Code
from aws_cdk.aws_events_targets import LambdaFunction
from aws_cdk.aws_events import Rule, EventPattern
from aws_cdk.aws_dynamodb import ITable
from aws_cdk.aws_events import IEventBus
from constructs import Construct

class LambdaUpdateStack(Stack):
    def __init__(self, scope: Construct, id: str, table: ITable, bus: IEventBus, detail_type: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        lambda_fn = Function(self, "UpdateLambda",
            runtime=Runtime.PYTHON_3_9,
            handler="index.handler",
            code=Code.from_asset("lambdas/update"),
            environment={"TABLE_NAME": table.table_name},
            timeout=Duration.seconds(10)
        )

        table.grant_write_data(lambda_fn)

        Rule(self, "UpdateRule",
            event_bus=bus,
            event_pattern=EventPattern(
                source=["crud.app"],
                detail_type=[detail_type]
            ),
            targets=[LambdaFunction(lambda_fn)]
        )


# ================================================
# File: stacks/lambda_delete_stack.py
# ================================================
from aws_cdk import Stack, Duration
from aws_cdk.aws_lambda import Function, Runtime, Code
from aws_cdk.aws_events_targets import LambdaFunction
from aws_cdk.aws_events import Rule, EventPattern
from aws_cdk.aws_dynamodb import ITable
from aws_cdk.aws_events import IEventBus
from constructs import Construct

class LambdaDeleteStack(Stack):
    def __init__(self, scope: Construct, id: str, table: ITable, bus: IEventBus, detail_type: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        lambda_fn = Function(self, "DeleteLambda",
            runtime=Runtime.PYTHON_3_9,
            handler="index.handler",
            code=Code.from_asset("lambdas/delete"),
            environment={"TABLE_NAME": table.table_name},
            timeout=Duration.seconds(10)
        )

        table.grant_write_data(lambda_fn)

        Rule(self, "DeleteRule",
            event_bus=bus,
            event_pattern=EventPattern(
                source=["crud.app"],
                detail_type=[detail_type]
            ),
            targets=[LambdaFunction(lambda_fn)]
        )


# =============================
# File: lambdas/create/index.py
# =============================
import json
import boto3
import os

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def handler(event, context):
    item = event['detail']
    table.put_item(Item=item)
    return {"statusCode": 200, "body": json.dumps("Item created")}


# ===========================
# File: lambdas/read/index.py
# ===========================
import json
import boto3
import os

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def handler(event, context):
    item_id = event['detail']['id']
    response = table.get_item(Key={"id": item_id})
    item = response.get("Item")
    return {"statusCode": 200, "body": json.dumps(item)}


# =============================
# File: lambdas/update/index.py
# =============================
import json
import boto3
import os

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def handler(event, context):
    item = event['detail']
    table.put_item(Item=item)
    return {"statusCode": 200, "body": json.dumps("Item updated")}


# =============================
# File: lambdas/delete/index.py
# =============================
import json
import boto3
import os

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def handler(event, context):
    item_id = event['detail']['id']
    table.delete_item(Key={"id": item_id})
    return {"statusCode": 200, "body": json.dumps("Item deleted")}


# =============================
# File: .github/workflows/deploy.yml
# =============================
name: CDK Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: CDK Bootstrap
        run: cdk bootstrap aws://$AWS_ACCOUNT_ID/$AWS_REGION
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_SESSION_TOKEN: ${{ secrets.AWS_SESSION_TOKEN }}

      - name: CDK Deploy
        run: cdk deploy --all --require-approval never
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_SESSION_TOKEN: ${{ secrets.AWS_SESSION_TOKEN }}
