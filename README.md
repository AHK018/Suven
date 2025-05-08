
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
import boto3
import os

def handler(event, context):
    print("Event received:", event)

    # Get table name from environment
    table_name = os.environ.get("TABLE_NAME")
    ddb = boto3.resource('dynamodb')
    table = ddb.Table(table_name)

    detail = event.get('detail', {})
    item = {
        "id": detail.get("id", "default-id"),
        "action": detail.get("action", "none")
    }

    # Write to DynamoDB
    table.put_item(Item=item)
    print("Item written to DynamoDB:", item)
"""),
            environment={
                "TABLE_NAME": table.table_name
            }
        )

        # Grant permissions
        table.grant_read_write_data(self.lambda_fn)
        


