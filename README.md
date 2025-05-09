
Step-by-Step Setup
✅ Step 1: In your central repo (e.g., aws-cred-repo)
📁 Path: .github/workflows/setup-aws.yml
yaml
Copy code
name: Setup AWS Credentials

on:
  workflow_call:
    inputs:
      region:
        required: true
        type: string

    secrets:
      AWS_ACCESS_KEY_ID:
        required: true
      AWS_SECRET_ACCESS_KEY:
        required: true
      AWS_SESSION_TOKEN:
        required: false

jobs:
  setup:
    runs-on: ubuntu-latest

    steps:
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }}
          aws-region: ${{ inputs.region }}
✅ Step 2: Add Secrets to aws-cred-repo
In aws-cred-repo → Settings → Secrets → Actions, add:

AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

AWS_SESSION_TOKEN (optional)

✅ OR BETTER: Store them as Organization Secrets if you want all repos in your org to access them without duplication.

✅ Step 3: In any other repo (consumer repo), use this central workflow
📁 Path: .github/workflows/deploy.yml
yaml
Copy code
name: Deploy App Using Central AWS Creds

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: your-org/aws-cred-repo/.github/workflows/setup-aws.yml@main
    with:
      region: us-east-1
    secrets: inherit

  cdk-deploy:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Install CDK
        run: npm install -g aws-cdk

      - name: Deploy with CDK
        run: cdk deploy --all --require-approval never
✅ What You Achieve
🔐 Central AWS credentials in aws-cred-repo

🔁 Reusable workflow for credential setup

📦 No need to manually pass secrets in every consumer repo

✅ Secure and GitHub-native

Would you like me to generate a ready-to-paste .zip with both example repos?







