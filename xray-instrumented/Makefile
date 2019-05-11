ENVIRONMENT ?= dev
STACK_NAME ?= sample-xray-instrumented-$(ENVIRONMENT)

all: build package deploy

# Build
CODEBUILD_START_TIME ?= "FALSE"
ifeq ($(CODEBUILD_START_TIME),"FALSE")
	BUILD_ARGS=--use-container
else
	BUILD_ARGS=
endif

build:
	sam build $(BUILD_ARGS)

# Package
package:
	sam package --s3-bucket $(S3_BUCKET) --template-file .aws-sam/build/template.yaml --output-template-file .aws-sam/build/template-output.yaml
	cp .aws-sam/build/template-output.yaml template-output.yml

# Deploy
deploy:
	sam deploy --template-file template-output.yml --stack-name $(STACK_NAME) --capabilities CAPABILITY_IAM

# Calls
calls:
	go run ../helpers/put.go $(shell aws cloudformation describe-stacks --stack-name $(STACK_NAME) --query 'Stacks[0].Outputs[?OutputKey==`ApiUrl`].OutputValue' --output text)
	go run ../helpers/get.go $(shell aws cloudformation describe-stacks --stack-name $(STACK_NAME) --query 'Stacks[0].Outputs[?OutputKey==`ApiUrl`].OutputValue' --output text)

# Local
local: build
	$(eval TABLE_NAME=$(shell aws cloudformation describe-stacks --stack-name $(STACK_NAME) --query 'Stacks[0].Outputs[?OutputKey==`TableName`].OutputValue' --output text))
	python3 ../helpers/gen_env.py TABLE_NAME=$(TABLE_NAME) > env.json
	@echo "Environment variables:"
	@cat env.json
	sam local start-api --template .aws-sam/build/template.yaml -n env.json

# Delete
delete:
	aws cloudformation delete-stack --stack-name $(STACK_NAME)
	aws cloudformation wait stack-delete-complete --stack-name $(STACK_NAME)