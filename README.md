# code workshop ec2 codedeploy

1. Download `cfn-infrastructure.yaml` from github, and deploy it in CloudFormation console, stack name: `cfn-code-workshop-ec2-deploy-stack`
2. Find Amazon Managed KMS Arn for s3 in the account, and paste it in the KmsKeyIdArn parameter, and select a key pair.
3. Deploy CloudFormation stack and wait for `CREATE_COMPLETE`

After successfully deployment:
1. Go to E2 Console, and get EC2's Public IP.
2. Open http://ec2-public-ip to test if you can see the test page
3. Then open url: http://ec2-public-ip/WordPress to setup WordPress
#### Database configuration:
* Database Name: `test`
* User Name: `root`
* Password: **Leave blank**.
* Database Host: localhost
* Table Prefix: wp_

4. Setup user, enter a dummy user, email etc. and remember password for later use.
#### Site configuration:
* Site Title: CodeDeployDemo
* Username: wordpress
* Password: wordpress
* Email: 123@test.com

5. Change theme: Login to WordPress, Appearance -> Theme -> search for `twentyfifteen` - **Activate**
6. Open url: http://ec2-public-ip/WordPress, you will see a new theme.
7. SSH into the EC2 instance, and publish a new package, do the following:
```
cd /tmp/WordPress

# replace color: #fff to #768331 for theme: twentyfifteen
sudo sed -i 's/#fff/#768331/g' wp-content/themes/twentyfifteen/style.css

# Publish changes to s3 bucket
aws deploy push \
    --application-name WordPress_App \
    --ignore-hidden-files \
    --region us-east-1 \
    --s3-location s3://codedeploydemo-117645918752-us-east-1-s3/WordPressApp.zip

# Create a new deployment
aws deploy create-deployment \
    --application-name WordPress_App \
    --deployment-config-name CodeDeployDefault.OneAtATime \
    --deployment-group-name CodeDeployDemoDeploymentGroup \
    --region us-east-1 \
    --s3-location bucket=codedeploydemo-117645918752-us-east-1-s3,bundleType=zip,key=WordPressApp.zip
```

8. Open incognito Window, and browse http://ec2-public-ip/WordPress.


## Reference:
[1] Deployment Configurations: `CodeDeployDefault.AllAtOnce`, `CodeDeployDefault.HalfAtATime`, and `CodeDeployDefault.OneAtATime`, 
https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-configurations.html#deployment-configurations-predefined


## Troubleshooting
When you create a new deployment, you notice the new deployment status shows in progress and all events shows in Pending, why?
* Status: In progress
<img width="938" alt="image" src="https://github.com/aws-k68pex/code-training-ec2-deploy/assets/104741984/bd18e572-d187-4634-8969-95d8c20f6795">
* Events shows in Pending:
<img width="939" alt="image" src="https://github.com/aws-k68pex/code-training-ec2-deploy/assets/104741984/f7f4b01e-00a6-4a81-82b6-7e92888102e1">

After a while, it will turn an Error below:
```
CodeDeploy agent was not able to receive the lifecycle event.
Check the CodeDeploy agent logs on your host and make sure the agent is running and can connect to the CodeDeploy server.
```

### Possibe reason:
1. CodeDeploy Agent is not installed or running,
2. EC2 is stopped or not existing,
3. Ec2TagFilters set incorrectly to cannot find correct EC2.
