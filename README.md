# acci-cf
Cloudformation templates for Astrocompute CI

# Templates

## Bootstrap

The first template is used to bootstrap the Astrocompute CI environment. Go into the directory where the repository is cloned and run:

    aws cloudformation create-stack \
		--stack-name astrocompute-ci-bootstrap \
		--template-body file://.//cloudformation//bootstrap.template \
		--parameters ParameterKey=BucketName,ParameterValue="cookbook-bucket"

Make sure the bucketname is unique, see http://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html

## opsworks-jenkins

This template is used to start an opsworks stack for setting up a jenkins server and some jenkins slaves:

    aws cloudformation create-stack --stack-name opsworks-jenkins \
		--template-body file://.//cloudformation//opsworks-jenkins-stack.template \
		--parameters ParameterKey=BucketName,ParameterValue="cookbook-bucket" \
		    ParameterKey=DomainName,ParameterValue="example.com", \
		    ParameterKey=ReplyToAddress,ParameterValue="me@example.com", \
			ParameterKey=KeyName,ParameterValue="aws_user_key" \
			ParameterKey="CookBookFile",ParameterValue="cookbooks.tar.gz"

The "BucketName" parameter points to the bucket that is created with the "bootstrap" template. The "DomainName" parameter is used for setting up the default suffix setting of the jenkins mailer plugin and CNAME record of the jenkins server. This must be a domainname that you control. The "ReplyToAddress" is the email address that is used to configure the reply-to setting in the jenkins mailer plugin. The "Keyname" parameter is the name of a ssh keypair, used to login to the created instances. The "CookBookFile" parameter is the name of the file of the custom Chef cookbook, used to configure the stacks instances. This tarball should be upload to the S3 bucket before creating the stack.

## Smtp

# Development

You can test the templates before using them by the command:

	aws cloudformation validate-template \
		--template-body file://.//cloudformation//bootstrap.template

This will return a json document if the template is correct.

## Chef cookbook

After a change in the custom cookbook of the stack, the cookbook should be packaged using the "berks" command and upload to the S3 bucket. After that, the new cookbook can be uploaded to the instances, using the following command:

	aws --region us-east-1 opsworks create-deployment \
	    --stack-id $(aws opsworks --region us-east-1 describe-stacks \
	        --query 'Stacks[?Name==`JenkinsStack`].[StackId]' --output text) \
	    --command "{\"Name\":\"update_custom_cookbooks\"}"

When the deployment is finished, execute the custom recipies using the following command:

	aws --region us-east-1 opsworks create-deployment \
		--stack-id $(aws opsworks --region us-east-1 describe-stacks \
			--query 'Stacks[?Name==`JenkinsStack`].[StackId]' --output text) \
		--command "{\"Name\":\"configure\"}"

# Cleanup

You can remove the stacks with the following commands:

	aws cloudformation delete-stack --stack-name opsworks-jenkins
	aws cloudformation delete-stack --stack-name astrocompute-ci-bootstrap

This removes all created resources from your account
