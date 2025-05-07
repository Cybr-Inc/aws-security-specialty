DEPLOYMENT INSTRUCTIONS:
  To deploy this template using the AWS CLI, run the following command:
  
  aws cloudformation deploy \
    --template-file aws-config-conforms-bucket-setup.yaml \
    --stack-name aws-config-conforms-bucket-stack \
    --parameter-overrides OrganizationId=o-yourorgid BucketSuffix=youruniquesuffix \
    --profile your-profile-name
  
  Replace:
  - 'o-yourorgid' with your actual AWS Organization ID
  - 'youruniquesuffix' with a unique identifier for your bucket
  - 'your-profile-name' with your AWS CLI profile name

  Retrieve the org id with:
  aws organizations describe-organization --query 'Organization.Id' --output text your-profile-name