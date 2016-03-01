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

## Smtp

# Development

You can test the templates before using them by the command:

	aws cloudformation validate-template --template-body file://.//cloudformation//bootstrap.template

This will return a json document if the template is correct.

# Cleanup

You can remove the stacks with the following commands:

	aws cloudformation delete-stack --stack-name astrocompute-ci-bootstrap

This removes all created resources from your account
