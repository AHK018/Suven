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
from stacks.custom_resource_stack import CustomResourceStack

app = App()

# Deploy stacks sequentially
dynamo_stack = DynamoDBStack(app, "DynamoDBStack")
eventbridge_stack = EventBridgeStack(app, "EventBridgeStack")

create_stack = LambdaCreateStack(app, "LambdaCreateStack",
    table=dynamo_stack.table,
    bus=eventbridge_stack.bus,
    detail_type="CREATE_ITEM")
create_stack.add_dependency(eventbridge_stack)

custom_resource_stack = CustomResourceStack(app, "CustomResourceStack",
    create_lambda=create_stack.create_lambda)
custom_resource_stack.add_dependency(create_stack)

read_stack = LambdaReadStack(app, "LambdaReadStack",
    table=dynamo_stack.table,
    bus=eventbridge_stack.bus,
    detail_type="READ_ITEM")
read_stack.add_dependency(custom_resource_stack)

update_stack = LambdaUpdateStack(app, "LambdaUpdateStack",
    table=dynamo_stack.table,
    bus=eventbridge_stack.bus,
    detail_type="UPDATE_ITEM")
update_stack.add_dependency(read_stack)

delete_stack = LambdaDeleteStack(app, "LambdaDeleteStack",
    table=dynamo_stack.table,
    bus=eventbridge_stack.bus,
    detail_type="DELETE_ITEM")
delete_stack.add_dependency(update_stack)

app.synth()


# ==============================
# File: data/items.csv
# ==============================
id,name,description
1,Item A,Description A
2,Item B,Description B
3,Item C,Description C


# ==============================
# File: lambdas/create_lambda/handler.py
# ==============================
import boto3
import csv
import os


dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def handler(event, context):
    with open("/var/task/data/items.csv", newline='') as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            table.put_item(Item=row)
    return {"statusCode": 200, "body": "Items inserted successfully"}


# ==============================
# File: lambdas/read_lambda/handler.py
# ==============================
import boto3
import os


dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def handler(event, context):
    item_id = event['detail']['id']
    response = table.get_item(Key={"id": item_id})
    return {"statusCode": 200, "body": response.get('Item', {})}


# ==============================
# File: lambdas/update_lambda/handler.py
# ==============================
import boto3
import os


dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def handler(event, context):
    detail = event['detail']
    table.update_item(
        Key={"id": detail['id']},
        UpdateExpression="SET #n = :name, description = :desc",
        ExpressionAttributeNames={"#n": "name"},
        ExpressionAttributeValues={":name": detail['name'], ":desc": detail['description']}
    )
    return {"statusCode": 200, "body": "Item updated"}


# ==============================
# File: lambdas/delete_lambda/handler.py
# ==============================
import boto3
import os


dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def handler(event, context):
    item_id = event['detail']['id']
    table.delete_item(Key={"id": item_id})
    return {"statusCode": 200, "body": "Item deleted"}


# ==============================
# File: stacks/custom_resource_stack.py
# ==============================
from aws_cdk import (
    Stack,
    custom_resources as cr,
)
from constructs import Construct

class CustomResourceStack(Stack):
    def __init__(self, scope: Construct, id: str, create_lambda, **kwargs):
        super().__init__(scope, id, **kwargs)

        provider = cr.Provider(
            self, "CustomResourceProvider",
            on_event_handler=create_lambda
        )

        cr.CustomResource(
            self, "TriggerCreateLambdaOnce",
            service_token=provider.service_token
        )
