# AWS Lambda-to-Lambda Invocation with Boto3

## Project Overview

This project demonstrates how two AWS Lambda functions communicate with each other using Python and Boto3.

The project was completed as part of my Cloud Computing & DevOps training at Montreal College of Information Technology (MCIT).

## Technologies Used

- AWS Lambda
- Python 3.13
- AWS SDK for Python (Boto3)
- AWS Identity and Access Management (IAM)
- Amazon CloudWatch
- JSON
- AWS Region: us-east-1 (N. Virginia)

## Project Architecture

**Test Event → MCIT-Caller → MCIT-Processor → MCIT-Caller → Response**

The project uses two AWS Lambda functions:

**MCIT-Caller:** Receives student information and invokes MCIT-Processor using Boto3.

**MCIT-Processor:** Validates student marks, calculates a grade, and returns the result.

The communication uses synchronous invocation (`RequestResponse`), meaning MCIT-Caller waits for MCIT-Processor to finish.

## Step 1 — Create MCIT-Processor

Created an AWS Lambda function named `MCIT-Processor` using Python 3.13.

The function receives student information and calculates a grade based on the student's marks.

### Python Code

```python
def lambda_handler(event, context):
    student_name = event.get("student_name", "Unknown")
    marks = event.get("marks", 0)

    if not isinstance(marks, (int, float)) or not 0 <= marks <= 100:
        raise ValueError("marks must be a number between 0 and 100")

    grade = "A" if marks >= 80 else "B" if marks >= 60 else "C" if marks >= 40 else "F"

    return {
        "student_name": student_name,
        "marks": marks,
        "grade": grade,
        "message": "Student processed successfully"
    }
```

## Step 2 — Test MCIT-Processor

Created a test event using the following JSON:

```json
{
  "student_name": "Sebastian",
  "marks": 95
}
```

**Result:** Execution succeeded and returned Grade A.

## Step 3 — Create MCIT-Caller

Created a second Lambda function named `MCIT-Caller`.

Configured the following environment variable:

```text
PROCESSOR_FUNCTION=MCIT-Processor
```

Set the Lambda timeout to 15 seconds.

## Step 4 — Configure IAM Permissions

Created an inline IAM policy named `MCIT-Invoke-Processor-Policy` and attached it to the MCIT-Caller execution role.

The policy allows MCIT-Caller to invoke MCIT-Processor using the `lambda:InvokeFunction` permission.

The permission is restricted to the MCIT-Processor Lambda function.

## Step 5 — Configure Lambda-to-Lambda Invocation

Added the following Python code to MCIT-Caller:

```python
import boto3
import json
import os

lambda_client = boto3.client("lambda")

def lambda_handler(event, context):
    payload = {
        "student_name": event.get("student_name", "Unknown"),
        "marks": event.get("marks", 0)
    }

    response = lambda_client.invoke(
        FunctionName=os.environ["PROCESSOR_FUNCTION"],
        InvocationType="RequestResponse",
        Payload=json.dumps(payload).encode("utf-8")
    )

    result = json.loads(response["Payload"].read().decode("utf-8"))

    if response.get("FunctionError"):
        raise RuntimeError(f"MCIT-Processor failed: {result}")

    return {
        "statusCode": 200,
        "body": result
    }
```

## Step 6 — Test Lambda-to-Lambda Communication

Executed MCIT-Caller using the following test event:

```json
{
  "student_name": "Sebastian",
  "marks": 95
}
```

### Successful Response

```json
{
  "statusCode": 200,
  "body": {
    "student_name": "Sebastian",
    "marks": 95,
    "grade": "A",
    "message": "Student processed successfully"
  }
}
```

Additional tests were completed to verify the grading system.

| Student Marks | Grade |
|---|---|
| 95 | A |
| 75 | B |
| 50 | C |
| 20 | F |

All tests completed successfully.

## Step 7 — Monitor with Amazon CloudWatch

Used Amazon CloudWatch to monitor both Lambda functions.

Verified:

- Lambda invocation logs
- START, END, and REPORT events
- Execution duration
- Memory usage
- Invocation metrics
- Error count and success rate

The MCIT-Caller metrics showed a 100% success rate and zero errors during the observed period.

## Synchronous vs. Asynchronous Invocation

**Synchronous (RequestResponse):** The calling Lambda waits for the target function to complete and receives its response.

**Asynchronous (Event):** The calling Lambda submits the event without waiting for the target function's processing result.

This project implemented and tested synchronous invocation.

## What I Learned

Through this project, I gained practical experience with:

- Creating and configuring AWS Lambda functions
- Using Boto3 to invoke AWS services
- Configuring least-privilege IAM permissions
- Passing JSON data between Lambda functions
- Using environment variables
- Understanding synchronous Lambda invocation
- Testing serverless applications
- Monitoring and troubleshooting with CloudWatch

## Project Status

**Completed — Successfully tested and verified.**

**Author:** Sebastian  
**Program:** Cloud Computing & DevOps — MCIT

##Screenshots##

<img width="2549" height="1266" alt="Screenshot 2026-10-29 201050" src="https://github.com/user-attachments/assets/4405d73a-da83-4a87-9f80-720e378969e7" />
<img width="2559" height="1311" alt="Screenshot 2026-10-29 200737" src="https://github.com/user-attachments/assets/937e21e1-a1ef-48f3-ab81-a7602ecea23a" />
<img width="2557" height="1306" alt="Screenshot 2026-10-29 200643" src="https://github.com/user-attachments/assets/86aa8ba8-6f00-4069-a2cc-b36b7491faf2" />
<img width="2559" height="1313" alt="Screenshot 2026-10-29 200617" src="https://github.com/user-attachments/assets/cfb2d2f5-62f2-45e4-b810-7ea57838d538" />
<img width="2558" height="1305" alt="Screenshot 2026-10-29 200553" src="https://github.com/user-attachments/assets/6e4f05a3-8a43-4d5e-822e-fd63d010baa6" />
<img width="2558" height="1307" alt="Screenshot 2026-10-29 200411" src="https://github.com/user-attachments/assets/10662747-e538-47f2-919a-9097aedd6feb" />
<img width="2559" height="1311" alt="Screenshot 2026-10-29 200256" src="https://github.com/user-attachments/assets/f79f6981-ece0-4e48-87ad-0918d6527331" />
<img width="2559" height="1306" alt="Screenshot 2026-10-29 195503" src="https://github.com/user-attachments/assets/7060d1fe-06fd-49e9-99de-8063d1a399e3" />
<img width="2558" height="1306" alt="Screenshot 2026-10-29 195214" src="https://github.com/user-attachments/assets/aed33243-f345-49ac-b0b5-113c1903d3b1" />
