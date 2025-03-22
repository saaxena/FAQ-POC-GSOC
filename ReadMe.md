# FAQ Assistant API Documentation

## Overview  (Please Go Through "Project-Overview.md" For Better Clarity )

This document provides instructions for testing the FAQ Assistant as part of the GSOC project proof of concept. The system consists of webhook endpoints that process questions from GitHub and Rocket.Chat, matching them against an FAQ database and responding according to configured workflows.

## Test Endpoints  (USE POSTMAN FOR TESTING EXPOSED ENDPOINTS )


### Health Check Endpoint

Method: GET
URL: https://faq-detect.onrender.com/health
This should return a simple status response to confirm the service is running.

### Test a GitHub Question

```
URL - https://faq-detect.onrender.com/test/webhook/github
Content-Type: application/json
```

**Request Body:**
```json
{
  "repository": {
    "full_name": "owner/repo1"
  },
  "issue": {
    "number": 123,
    "body": "How do I set up the development environment for this project?"
  },
  "sender": {
    "login": "test-user"
  }
}
```

**Response:**
```json
{
  "status": "success",
  "message": "Test webhook processed successfully",
  "details": {
    "source": "github",
    "question": "How do I set up the development environment for this project?",
    "context": {
      "source": "github",
      "repo": "owner/repo1",
      "issueNumber": 123,
      "questionerId": "test-user"
    }
  }
}
```

### Test a Rocket.Chat Question

```
URL - https://faq-detect.onrender.com/test/webhook/rocketchat
Content-Type: application/json
```

**Request Body:**
```json
{
  "text": "How do I submit a pull request?",
  "channel_name": "contributors",
  "user_name": "test-user"
}
```

**Response:**
```json
{
  "status": "success",
  "message": "Test webhook processed successfully",
  "details": {
    "source": "rocketchat",
    "question": "How do I submit a pull request?",
    "context": {
      "source": "rocketchat",
      "channel": "contributors",
      "questionerId": "test-user"
    }
  }
}
```

### Test the Approval Workflow

```
URL: https://faq-detect.onrender.com/test/approval
Content-Type: application/json
```

**Request Body:**
```json
{
  "answerId": "answer-1234",
  "approved": true,
  "userId": "maintainer1",
  "modifiedAnswer": "Optional modified answer if edited by maintainer"
}
```

**Response:**
```json
{
  "status": "success",
  "message": "Answer approved successfully",
  "details": {
    "answerId": "answer-1234",
    "approved": true,
    "userId": "maintainer1",
    "timestamp": "2025-03-22T15:30:45.123Z"
  }
}
```

## Configuration

The system's behavior is controlled by the `settings.json` file which determines:

1. **Action Type**: How the system responds to matched questions
   - `DIRECT_ANSWER`: Automatically post answers
   - `DM_WITH_APPROVAL`: Send to maintainers for approval before posting
   - `NOTIFY_ONLY`: Only notify maintainers about matches

2. **Matching Threshold**: Confidence level required for a question to match an FAQ (0-1)

3. **Notify Users**: List of maintainers who receive notifications/approvals

## Testing with cURL

Test GitHub question matching:
```bash
curl -X POST https://faq-detect.onrender.com/test/webhook/github \
  -H "Content-Type: application/json" \
  -d '{"repository":{"full_name":"owner/repo1"},"issue":{"number":123,"body":"How do I set up the development environment for this project?"},"sender":{"login":"test-user"}}'
```

Test Rocket.Chat question matching:
```bash
curl -X POST https://faq-detect.onrender.com/test/webhook/rocketchat \
  -H "Content-Type: application/json" \
  -d '{"text":"How do I submit a pull request?","channel_name":"contributors","user_name":"test-user"}'
```

Test approval process:
```bash
curl -X POST https://faq-detect.onrender.com/test/approval \
  -H "Content-Type: application/json" \
  -d '{"answerId":"answer-1234","approved":true,"userId":"maintainer1"}'
```

## Logs

The system logs all actions, which can be viewed in the console during testing. Look for:

1. Question processing
2. FAQ matching results
3. Answer generation
4. Approval requests
5. Final actions taken

These logs provide visibility into the system's decision-making process during demos.