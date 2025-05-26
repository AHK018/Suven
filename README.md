name: CDK Deploy and Drift Detection

on:
  push:
    branches: [main]  # or your deployment branch

jobs:
  deploy-and-drift-check:
    runs-on: ubuntu-latest

    env:
      AWS_REGION: us-east-1  # adjust your region

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Install dependencies
      run: |
        npm install -g aws-cdk
        python -m pip install -r requirements.txt  # or npm install for JS projects

    - name: Deploy CDK stacks
      run: |
        cdk deploy --require-approval never --outputs-file cdk-outputs.json
        cdk list > deployed-stacks.txt

    - name: Drift Detection
      run: |
        while read stack; do
          echo "Starting drift detection for $stack..."
          DETECTION_ID=$(aws cloudformation detect-stack-drift --stack-name "$stack" --query 'StackDriftDetectionId' --output text)
          echo "Waiting for drift detection to complete for $stack..."
          STATUS="DETECTION_IN_PROGRESS"
          while [ "$STATUS" == "DETECTION_IN_PROGRESS" ]; do
            sleep 5
            STATUS=$(aws cloudformation describe-stack-drift-detection-status --stack-drift-detection-id "$DETECTION_ID" --query 'DetectionStatus' --output text)
          done
          DRIFT_STATUS=$(aws cloudformation describe-stack-drift-detection-status --stack-drift-detection-id "$DETECTION_ID" --query 'StackDriftStatus' --output text)
          echo "$stack: $DRIFT_STATUS"
          if [ "$DRIFT_STATUS" != "IN_SYNC" ]; then
            echo "Drift detected in stack: $stack"
            exit 1
          fi
        done < deployed-stacks.txt
