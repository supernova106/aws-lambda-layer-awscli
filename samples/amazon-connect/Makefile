ROOT ?= $(shell pwd)
AWS_ACCOUNT_ID := $(shell aws sts get-caller-identity --query 'Account' --output text)
LAYER_NAME ?= awscli-layer
LAYER_DESC ?= awscli-layer
S3BUCKET ?= pahud-tmp-ap-northeast-1
LAMBDA_REGION ?= ap-northeast-1
LAMBDA_FUNC_NAME ?= awscli-version
LAMBDA_FUNC_DESC ?= awscli-version


.PHONY: prep	
prep:
	chmod +x main.sh bootstrap
	( rm -rf ./func.d; mkdir ./func.d; cp main.sh bootstrap ./func.d)
	

.PHONY: invoke
invoke:
	@aws --region $(LAMBDA_REGION) lambda invoke --function-name $(LAMBDA_FUNC_NAME)  \
	--payload "" lambda.output --log-type Tail | jq -r .LogResult | base64 -d	

.PHONY: sam-package
sam-package:
	@docker run -ti \
	-v $(PWD):/home/samcli/workdir \
	-v $(HOME)/.aws:/home/samcli/.aws \
	-w /home/samcli/workdir \
	-e AWS_DEFAULT_REGION=$(LAMBDA_REGION) \
	pahud/aws-sam-cli:latest sam package --template-file sam.yaml --s3-bucket $(S3BUCKET) --output-template-file packaged.yaml


.PHONY: sam-deploy
sam-deploy:
	@docker run -ti \
	-v $(PWD):/home/samcli/workdir \
	-v $(HOME)/.aws:/home/samcli/.aws \
	-w /home/samcli/workdir \
	-e AWS_DEFAULT_REGION=$(LAMBDA_REGION) \
	pahud/aws-sam-cli:latest sam deploy --template-file ./packaged.yaml --stack-name "$(LAMBDA_FUNC_NAME)-stack" --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND
	# print the cloudformation stack outputs
	aws --region $(LAMBDA_REGION) cloudformation describe-stacks --stack-name "$(LAMBDA_FUNC_NAME)-stack" --query 'Stacks[0].Outputs'

.PHONY: sam-publish
sam-publish:
	@docker run -ti \
	-v $(PWD):/home/samcli/workdir \
	-v $(HOME)/.aws:/home/samcli/.aws \
	-w /home/samcli/workdir \
	-e AWS_DEFAULT_REGION=$(LAMBDA_REGION) \
	pahud/aws-sam-cli:latest sam publish --region $(LAMBDA_REGION) --template packaged.yaml
	


.PHONY: sam-destroy
sam-destroy:
	# destroy the stack	
	aws --region $(LAMBDA_REGION) cloudformation delete-stack --stack-name "$(LAMBDA_FUNC_NAME)-stack"

.PHONY: clean
clean:
	rm -rf awscli-bundle* layer layer.zip func-bundle.zip lambda.output
	

.PHONY: clean-all
clean-all: clean
	@aws --region $(LAMBDA_REGION) lambda delete-function --function-name $(LAMBDA_FUNC_NAME)
	
	
