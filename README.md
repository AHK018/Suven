
name: Create or Update Lambda Function

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    env:
      FUNCTION_NAME: my-github-lambda
      HANDLER: lambda_function.lambda_handler
      RUNTIME: python3.9
      ROLE_ARN: arn:aws:iam::YOUR_ACCOUNT_ID:role/YOUR_LAMBDA_EXECUTION_ROLE

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.9

      - name: Install dependencies and zip
        run: |
          mkdir -p package
          pip install -r requirements.txt -t package
          cp lambda_function.py package/
          cd package && zip -r ../lambda.zip .

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Create or Update Lambda Function
        run: |
          set +e
          aws lambda get-function --function-name $FUNCTION_NAME
          EXISTS=$?
          set -e

          if [ $EXISTS -eq 0 ]; then
            echo "Updating existing Lambda function..."
            aws lambda update-function-code \
              --function-name $FUNCTION_NAME \
              --zip-file fileb://lambda.zip
          else
            echo "Creating new Lambda function..."
            aws lambda create-function \
              --function-name $FUNCTION_NAME \
              --runtime $RUNTIME \
              --role $ROLE_ARN \
              --handler $HANDLER \
              --zip-file fileb://lambda.zip
          fi


