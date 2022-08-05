## Welcome to the Innovator Island workshop!

![InnovatorIsland](./images/innovator-island_logo.png)

For workshop instructions, visit the [workshop's new instructions site](https://www.eventbox.dev/published/lesson/innovator-island/)

### Have an idea for this workshop? Found a bug? ###

If you have an idea for a module or feature in this workshop, or you have found a bug or need to report a problem, let us know!

- [Request a feature](https://github.com/aws-samples/aws-serverless-workshop-innovator-island/issues/new?assignees=&labels=&template=workshop-feature-request.md&title=)
- [Report an issue](https://github.com/aws-samples/aws-serverless-workshop-innovator-island/issues/new?assignees=&labels=&template=bug_report.md&title=)

AWS_REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/')

cd ~/environment/
git clone https://github.com/aws-samples/aws-serverless-workshop-innovator-island ./theme-park-backend

sudo yum install jq -y


accountId=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .accountId)
s3_deploy_bucket="theme-park-sam-deploys-${accountId}"
aws s3 mb s3://$s3_deploy_bucket

cd ~/environment/theme-park-backend/1-app-deploy/sam-app/
sam build
sam package --output-template-file packaged.yaml --s3-bucket $s3_deploy_bucket
sam deploy --template-file packaged.yaml --stack-name theme-park-backend --capabilities CAPABILITY_IAM

cd ~/environment/theme-park-backend/1-app-deploy/ride-controller/
sam package --output-template-file packaged.yaml --s3-bucket $s3_deploy_bucket
sam deploy --template-file packaged.yaml --stack-name theme-park-ride-times --capabilities CAPABILITY_IAM

AWS_REGION=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/')
FINAL_BUCKET=$(aws cloudformation describe-stack-resource --stack-name theme-park-backend --logical-resource-id FinalBucket --query "StackResourceDetail.PhysicalResourceId" --output text)
PROCESSING_BUCKET=$(aws cloudformation describe-stack-resource --stack-name theme-park-backend --logical-resource-id ProcessingBucket --query "StackResourceDetail.PhysicalResourceId" --output text)
UPLOAD_BUCKET=$(aws cloudformation describe-stack-resource --stack-name theme-park-backend --logical-resource-id UploadBucket --query "StackResourceDetail.PhysicalResourceId" --output text)
DDB_TABLE=$(aws cloudformation describe-stack-resource --stack-name theme-park-backend --logical-resource-id DynamoDBTable --query "StackResourceDetail.PhysicalResourceId" --output text)
echo $FINAL_BUCKET
echo $PROCESSING_BUCKET
echo $UPLOAD_BUCKET
echo $DDB_TABLE

cd ~/environment/theme-park-backend/1-app-deploy/local-app/
npm install
node ./importData.js $AWS_REGION $DDB_TABLE

aws dynamodb scan --table-name $DDB_TABLE
aws cloudformation describe-stacks --stack-name theme-park-backend --query "Stacks[0].Outputs[?OutputKey=='InitStateApi'].OutputValue" --output text



