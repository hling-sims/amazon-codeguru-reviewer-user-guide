# Create code reviews with GitHub Actions<a name="working-with-cicd"></a>

Amazon CodeGuru Reviewer finds issues in your Java and Python code and recommends how to remediate them\. CodeGuru Reviewer detects deviation from best practices for using AWS APIs and SDKs, and also identifies concurrency issues, resource leaks, security vulnerabilities and validates input parameters validation\. First, you enable CodeGuru Reviewer on your build workflow\. Next, for every push, pull, or scheduled repository scan, the CodeGuru Reviewer GitHub Action copies your code and build artifacts into an S3 bucket in your AWS account\. CodeGuru Reviewer APIs are used to analyze the artifacts and provide recommendations\.

You can enable security and code quality recommendations with GitHub Actions by making the following changes to your workflow\. If your repository has files in both Java and Python, then CodeGuru Reviewer will provide recommendations for the language that has more files\. An example workflow file could be `.github/workflows/build.yml`\.

## Get recommendations using GitHub Actions<a name="working-with-github-actions"></a>

This sections shows you how to create recommendations using GitHub Actions and disassociate a workflow, and also provides examples to get your started\. 

**Topics**
+ [Create code reviews with GitHub Actions](#create-recommendations-with-github-actions)
+ [Disassociate your CI/CD workflow](#disassociate-your-workflow)
+ [GitHub Actions code review examples](#codeguru-reviewer-on-github-hosted-runner-example)

### Create code reviews with GitHub Actions<a name="create-recommendations-with-github-actions"></a>

This section shows you how to create code reviews and get recommendations using GitHub Actions\.

**To start recommendations using GitHub Actions**

1. Create an Amazon S3 bucket with the prefix **codeguru\-reviewer\-\*** to upload your code and artifacts\. For information on creating a new Amazon S3 bucket, see [Creating a bucket](https://docs.aws.amazon.com/amazonS3/latest/userguide/create-bucket-overview.html) in the *Amazon S3 User Guide*\.

1. Sign into your GitHub account to complete the CI/CD integration process\. Your repository must be public or private if it's part of a GitHub organization in order for GitHub actions to work\.

1. In order to run the CodeGuru Reviewer Action, you need to provide AWS credentials\. We recommend using [aws\-actions/configure\-aws\-credentials](https://github.com/aws-actions/configure-aws-credentials) to configure your credentials for a job\. For self\-hosted runners, the `configure-aws-credentials` action assumes the runner’s IAM credentials or role to the CodeGuru Reviewer Action\. Docker must be installed for self\-hosted runners\. For information on installing Docker, see [Get started with Docker](https://docs.docker.com/get-started/)\.

   For GitHub hosted runners, you can configure the credentials in GitHub Secrets\. 

   The IAM user or IAM role should have the `AmazonCodeGuruReviewerFullAccess` policy enabled and Amazon S3 Permissions `(s3:PutObject, s3:ListBucket, s3:GetObject)`\. For more details on AWS credentials, see [Configuration and credential file settings](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) in the *AWS Command Line Interface User Guide*\. 

1. Add the CodeGuru Reviewer Action\. The following code snippet provides an example showing how you can enable your workflow, as supported by CodeGuru Reviewer\.

   ```
   - name: Amazon CodeGuru Reviewer Scanner
         if: ${{ always() }}
         uses: aws-actions/codeguru-reviewer@v1.1
         with:
           build_path: target # build artifact(s) directory. This is only required for Java repositories
           s3_bucket: codeguru-reviewermyactions-bucket  # S3 Bucket with "codeguru-reviewer-*" prefix
   ```

   The following is a list of parameters\.    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/codeguru/latest/reviewer-ug/working-with-cicd.html)

1. Run your workflow in GitHub to start the code analysis\. When the build is complete, review your recommendations in the GitHub **Security** tab\.

### Disassociate your CI/CD workflow<a name="disassociate-your-workflow"></a>

If your CI workflow association fails, you can disassociate your repository by choosing **Disassociate repository**\. If you want to associate your CI workflow later, you can associate your repository again by following set up steps\.

If you want to stop CodeGuru Reviewer recommendations for your CI workflow, remove the `codeguru` action script from your repository’s YML file\. Then, choose **Disassociate repository** to remove the repository association\. On your next job run, CodeGuru Reviewer associates the repository again unless you remove the `codeguru` action script from the YML file\.

### GitHub Actions code review examples<a name="codeguru-reviewer-on-github-hosted-runner-example"></a>

Run CodeGuru Reviewer Action on a GitHub hosted runner\.

```
steps:
    - name: Checkout repository
      uses: actions/checkout@v2
      with:
        fetch-depth: 0   # Required
   
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v1
      if: ${{ always() }} # This ensures that your workflow runs successfully
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-west-2

    - name: Amazon CodeGuru Reviewer Scanner
      uses: aws-actions/codeguru-reviewer@v1.1
      if: ${{ always() }} 
      with:
        build_path: target # build artifact(s) directory
        s3_bucket: codeguru-reviewer-my-bucket  # S3 Bucket with "codeguru-reviewer-*" prefix
    
    - name: Upload review result
      if: ${{ github.event_name != 'push' }}
      uses: github/codeql-action/upload-sarif@v1
      with:
        sarif_file: codeguru-results.sarif.json
```

Run a CodeGuru Reviewer Action on a self\-hosted runner\.

```
steps:
    - name: Checkout repository
      uses: actions/checkout@v2
      with:
        fetch-depth: 0

    - name: Configure AWS Credentials
      if: ${{ always() }} #  This ensures that your workflow runs successfully
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-region: us-west-2
        role-to-assume: my-github-actions-role

    # Refer to Step 2 for more details
    - name: Amazon CodeGuru Reviewer
      uses: aws-actions/codeguru-reviewer@v1.1
      if: ${{ always() }}
      with:          
        build_path: target # build artifact(s) directory
        s3_bucket: codeguru-reviewermy-bucket

    - name: Upload review result
      if: ${{ github.event_name != 'push' }}
      uses: github/codeql-action/upload-sarif@v1
      with:
        sarif_file: codeguru-results.sarif.json
```