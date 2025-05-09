# CI/CD Pipeline for AWS Lambda Deployment

This repository contains the Jenkins pipeline (`Jenkinsfile`) that automates the CI/CD process for deploying an AWS Lambda function. The Lambda function's source code and dependencies are stored in a separate GitHub repository.

---

## 🔧 Project Structure

- **Repo A** – Holds the AWS Lambda function code and its `requirements.txt`.
- **Repo B (this repo)** – Contains the Jenkinsfile and automates:
  - Cloning the Lambda code from Repo A
  - Packaging the Lambda function
  - Deploying it to AWS

---

## 🚀 Pipeline Overview

The `Jenkinsfile` defines the following stages:

### 1. **Clone Lambda Repo**
- Clones the Lambda code from **Repo A**.
- Uses the specified branch (default is `main`).
- The code is cloned into a subdirectory called `lambda-function`.

```groovy
dir('lambda-function') {
    git branch: "${env.LAMBDA_REPO_BRANCH}", url: "${env.LAMBDA_REPO_URL}"
}
2. Install Dependencies & Package
Installs Python dependencies listed in requirements.txt.

Copies Python source files into a build folder.

Zips the content into function.zip, which is required for AWS Lambda deployment.

mkdir -p build
pip install -r requirements.txt -t build/
cp *.py build/
cd build
zip -r ../function.zip .
💡 If your Python files are in a subfolder, modify the cp command accordingly.

3. Deploy to AWS Lambda
Authenticates with AWS using credentials configured in Jenkins.

Uses the AWS CLI to update the Lambda function's code using the function.zip file.

aws lambda update-function-code \
    --function-name ${LAMBDA_FUNCTION_NAME} \
    --zip-file fileb://function.zip
🔐 Prerequisites
Jenkins Agent Requirements

Python 3.x installed

pip installed

awscli installed and available in PATH

AWS Credentials

Configure AWS credentials in Jenkins using the AWS Credentials plugin

Replace aws-credentials-id in the Jenkinsfile with your credential ID

🔄 Triggering the Pipeline
This pipeline is currently stored in Repo B and manually or automatically triggered based on your Jenkins setup.

If you want the pipeline to auto-trigger on changes in Repo A, consider one of the following:

Use GitHub Webhooks and write a polling job

Use a multi-repo monorepo setup

Use an external trigger script

📁 Environment Variables
Variable Name	Description
LAMBDA_REPO_URL	URL to the Lambda code GitHub repository (Repo A)
LAMBDA_REPO_BRANCH	Git branch to clone (default: main)
LAMBDA_FUNCTION_NAME	Name of the Lambda function in AWS

✅ Success and Failure Notifications
At the end of the pipeline:

If deployment is successful, a message Lambda deployment successful! is shown.

If it fails, the pipeline prints Lambda deployment failed.

📌 Notes
Adjust the cp and zip logic if your Lambda handler is in a nested directory.

Add additional stages for:

Code linting (e.g., flake8)

Unit testing (e.g., pytest)

Integration with GitHub PR checks
