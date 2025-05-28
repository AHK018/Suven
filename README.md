Bad schema error in Tavern usually means the YAML test file has a formatting or schema issue — for example, using unsupported fields, incorrect nesting, or misconfigured hooks.

Let me guide you through a working example where we import AWS credentials from AWSauthhook.py and attach AWS4Auth to the request using a custom request hook, which is the correct and supported way in Tavern.

✅ Step-by-step Working Example
1. ✅ AWSauthhook.py (define the request hook)
This file contains the hook function that Tavern will call:

python
Copy code
# AWSauthhook.py

from requests_aws4auth import AWS4Auth
import boto3

def modify_request_with_aws_auth(request_args, test_case_dict, test_block_config, variables):
    """
    Hook to attach AWS auth to the request.
    """
    session = boto3.Session()
    credentials = session.get_credentials().get_frozen_credentials()
    region = 'us-west-2'  # change this to your region
    service = 'execute-api'  # change this to your service, like 's3', 'es', etc.

    awsauth = AWS4Auth(
        credentials.access_key,
        credentials.secret_key,
        region,
        service,
        session_token=credentials.token
    )

    request_args["auth"] = awsauth
    return request_args
2. ✅ test_aws_request.tavern.yaml (use the hook)
yaml
Copy code
test_name: "Test AWS API with AWS4Auth"

stages:
  - name: "Send signed request to AWS endpoint"
    request:
      url: "https://your-api-id.execute-api.us-west-2.amazonaws.com/dev/endpoint"
      method: GET
      headers:
        content-type: application/json
      # Hook to modify request and inject auth
      hooks:
        request:
          - AWSauthhook:modify_request_with_aws_auth

    response:
      status_code: 200
❗ IMPORTANT:

Ensure the indentation is correct.

The hook is under request -> hooks -> request -> - AWSauthhook:modify_request_with_aws_auth.

3. ✅ Folder Structure
Copy code
project/
│
├── AWSauthhook.py
└── test_aws_request.tavern.yaml
Make sure you're running the test from the same directory, or PYTHONPATH includes the hook location.

4. ✅ Install Dependencies
bash
Copy code
pip install tavern requests-aws4auth boto3
5. ✅ Run the Test
bash
Copy code
tavern-ci test_aws_request.tavern.yaml
🔍 If You Still See Bad schema:
Check YAML syntax: use 2 spaces for indentation (not tabs).

Ensure AWSauthhook.py is importable (i.e., in same folder or in Python path).

Make sure your hook is returning a modified request_args dict.

Don't put hooks directly under the top-level request: (only under request.hooks.request).

Let me know your Tavern/YAML file and folder structure if you're still getting errors — I’ll debug it directly.






Is this conversation helpful so far?






Tools



ChatGPT can make mistakes. Check 