up:
	terraform apply --auto-approve -var environment=${ENV}

down:
	terraform destroy --auto-approve -var environment=${ENV}

init:
	terraform init --backend-config="key=state/${ENV}.tfstate" --backend-config="dynamodb_table=$(shell cd /environment/infra && terraform output dynamoDb_lock_table_name)" --backend-config="bucket=$(shell cd /environment/infra && terraform output state_bucket_name)"
