# SNS Message Data Protection Instructions

## Resources Created by the Template

1. **SNS Topic** with data protection policy
2. **CloudWatch Logs Group** for audit findings
3. **Email Subscription** to the topic
4. **Lambda Function** to generate test transactions
5. **IAM Role** for Lambda to publish to SNS

## The Data Protection Policy

The CloudFormation template implements a policy that:

1. **Audits Inbound Messages**:
   - Detects credit card numbers in messages being published to the topic
   - Logs findings to CloudWatch (for monitoring and compliance)
   - Maintains the original message content for processing

2. **Masks Outbound Messages**:
   - For any credit card numbers that might still be in the message
   - Replaces digits with "X" characters (XXXXXXXXXXXXXXXX)
   - Applies to all outbound messages to subscribers

## Deployment Instructions

### Option 1: Using the AWS Management Console

1. Sign in to the AWS Management Console
2. Navigate to the CloudFormation service
3. Click "Create stack" > "With new resources"
4. Upload the `sns-data-protection-template.yaml` file
5. Provide parameters:
   - `EmailSubscriber`: Your email address to receive notifications
   - `TopicName`: (Optional) Custom name for the SNS topic
6. Complete the stack creation wizard (Next > Next > Create stack)
7. Wait for the stack to reach the CREATE_COMPLETE status

### Option 2: Using the AWS CLI

1. Ensure you have the AWS CLI installed and configured with appropriate credentials
2. Save the `sns-data-protection-template.yaml` file to your local machine
3. Open a terminal and run the following command:

   ```bash
   aws cloudformation create-stack \
     --stack-name SNSDataProtectionLab \
     --template-body file://path/to/sns-data-protection-template.yaml \
     --parameters \
         ParameterKey=EmailSubscriber,ParameterValue=your.email@example.com \
         ParameterKey=TopicName,ParameterValue=CreditCardProtectionTopic \
     --capabilities CAPABILITY_IAM
   ```
   (Don't forget your --profile <name> if needed)

   Replace `path/to/sns-data-protection-template.yaml` with the actual path to the file and `your.email@example.com` with your actual email address.

4. To monitor the stack creation status:

   ```bash
   aws cloudformation describe-stacks --stack-name SNSDataProtectionLab
   ```

5. Once deployed, you can get the outputs (like the Lambda function ARN and SNS topic ARN):

   ```bash
   aws cloudformation describe-stacks \
     --stack-name SNSDataProtectionLab \
     --query "Stacks[0].Outputs"
   ```

## Testing the Setup

1. **Confirm your email subscription**:
   - You'll receive a confirmation email
   - Click the "Confirm subscription" link

2. **Basic Testing with Lambda**:

   ```bash
   # For AWS CLI v1 or if you've configured CLI binary format:
   aws lambda invoke --function-name TestCreditCardTransaction --payload '{"cc_number":"4111111111111111"}' response.json
   
   # For AWS CLI v2 (recommended):
   aws lambda invoke --function-name TestCreditCardTransaction --payload '{"cc_number":"4111111111111111"}' --cli-binary-format raw-in-base64-out response.json
   
   # Alternative file-based approach (most reliable):
   echo '{"cc_number":"4111111111111111"}' > payload.json
   aws lambda invoke --function-name TestCreditCardTransaction --payload file://payload.json response.json
   ```

   > **Note**: AWS CLI v2 changed how binary payloads are handled, requiring the `--cli-binary-format` flag.
   > To make this setting permanent: `aws configure set cli-binary-format raw-in-base64-out`

   Or use the AWS Console:
   - Navigate to the Lambda service
   - Find and select "TestCreditCardTransaction"
   - Create a test event with payload: `{"cc_number":"4111111111111111"}`
   - Click "Test"

3. **Verify Results**:
   - **Check CloudWatch Logs**:
     - Navigate to CloudWatch > Log groups > "/aws/vendedlogs/sns-data-protection-audit"
     - Look for entries showing credit card detection
   
   - **Check your email**:
     - You should receive an email with the transaction
     - The credit card number should appear as XXXXXXXXXXXXXXXX

## Understanding What's Happening

1. The Lambda function generates a transaction with a credit card number
2. When the message is published to SNS:
   - The inbound data protection rule detects the credit card number
   - An audit log is created in CloudWatch for compliance purposes
3. When SNS delivers the message to subscribers:
   - The outbound data protection rule applies masking
   - Email subscribers see the masked credit card number

## Cleanup

### Option 1: Using the AWS Management Console
To avoid ongoing charges, delete the CloudFormation stack when you're done:
1. Navigate to CloudFormation in the AWS Console
2. Select the stack you created
3. Click "Delete" and confirm
4. Wait for the stack deletion to complete

### Option 2: Using the AWS CLI
```bash
aws cloudformation delete-stack --stack-name SNSDataProtectionLab
```

To check the deletion status:
```bash
aws cloudformation describe-stacks --stack-name SNSDataProtectionLab
```
Once the stack is completely deleted, this command will return an error stating the stack doesn't exist.
