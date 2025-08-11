# QA Tests Automation

![Architecture Diagram](qa_tests_automation_diagram.png)

## Overview

This project provides AWS infrastructure to automate regression test execution whenever developers push changes to production or staging environments. It runs comprehensive API tests using Mocha framework to verify that all endpoints continue working as expected after deployments, immediately alerting teams when changes break existing functionality.

## Architecture

The solution uses the following AWS services:

-   **AWS CodeBuild**: Executes QA tests in a containerized environment
-   **Amazon S3**: Stores test reports as a static website for easy access
-   **Amazon SNS**: Sends email notifications with test results
-   **AWS Lambda**: Processes build events and formats notifications
-   **Amazon EventBridge**: Triggers notifications based on build status

## Features

-   ✅ Automated regression test execution on deployments
-   ✅ Comprehensive API endpoint validation
-   ✅ Mocha-based test framework with Mochawesome reporting
-   ✅ Email notifications to QA team, developers, and scrum masters
-   ✅ Web-accessible HTML test reports with detailed results
-   ✅ Environment-specific test execution (staging/production)
-   ✅ Integration with GitHub Actions CI/CD pipeline
-   ✅ Automatic test skipping for production-restricted scenarios

## Prerequisites

-   AWS CLI configured with appropriate permissions
-   SAM CLI installed
-   Node.js 22+ for test execution
-   GitHub repository with Mocha test suite
-   Mochawesome configured for HTML report generation

## Deployment

1. **Clone the repository**

    ```bash
    git clone <repository-url>
    cd QA_Tests_Automation
    ```

2. **Configure test repository**
   Update the GitHub repository URL in `template.yaml` where your test suite is located:

    ```yaml
    # In template.yaml, update the BuildProject Source Location
    Source:
        Type: GITHUB
        Location: https://github.com/<your-username>/<your-test-repo>
    ```

3. **Deploy the infrastructure**

    ```bash
    sam build
    sam deploy --guided
    ```

4. **Confirm email subscriptions**
    - Confirm email subscriptions from AWS SNS via AWS Console

## Configuration

### Environment Variables

-   `COMMAND`: Test command to execute (e.g., `test.staging`, `test.production`)
-   `REPORTS_BUCKET`: S3 bucket for storing Mochawesome HTML reports

### Test Environments

-   **Staging**: Full regression test suite execution
-   **Production**: Selective test execution (some tests skipped for safety)

### Test Integration

Add this step to your GitHub Actions workflow:

```yaml
- name: Run QA tests
  run: |
      BUILD_ID=$(aws codebuild start-build \
        --project-name qa-tests-automation \
        --environment-variables-override name=COMMAND,value=test.example \
        --query 'build.id' --output text)

      # Wait for completion and handle results
      while true; do
        STATUS=$(aws codebuild batch-get-builds --ids $BUILD_ID \
          --query 'builds[0].buildStatus' --output text)

        if [ "$STATUS" = "SUCCEEDED" ]; then
          break
        elif [ "$STATUS" = "FAILED" ]; then
          exit 1
        fi
        sleep 30
      done
```

## Usage

1. **Automatic Trigger**: Regression tests run automatically after deployments
2. **Manual Trigger**: Start builds via AWS Console or CLI for ad-hoc testing
3. **View Reports**: Access detailed Mochawesome HTML reports via S3 website
4. **Monitor Results**: QA team, developers, and scrum masters receive email notifications
5. **Failure Analysis**: Identify which APIs broke due to recent changes
6. **Environment Selection**: Configure different test suites for staging vs production

## File Structure

```
├── template.yaml          # SAM template with AWS resources
├── buildspec.yml          # CodeBuild build specification
├── README.md              # This file
└── samconfig.toml         # SAM configuration
```
