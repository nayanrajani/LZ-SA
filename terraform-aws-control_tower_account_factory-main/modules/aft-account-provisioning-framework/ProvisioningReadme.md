# aft-account-provisioning-framework
- Let's start with main.tf file
    - <img width="688" alt="image" src="https://user-images.githubusercontent.com/57224583/196680190-c49ef6a9-b3ef-4333-9868-38e97edc8505.png">

    - module "aft_account_provisioning_framework" {
        providers = {
            aws = aws.aft_management
        }
        - source                                           = "./modules/aft-account-provisioning-framework"
            - this actually is the folder where we are creating the readme file.
        - aft_account_provisioning_framework_sfn_name      = local.aft_account_provisioning_framework_sfn_name
            - this is the local file of the main folder where it will access the fsn name i.e., "aft-account-provisioning-framework".
        - aft_account_provisioning_customizations_sfn_name = local.aft_account_provisioning_customizations_sfn_name
            - this is the local file of the main folder where it will access the fsn name i.e., "aft-account-provisioning-customizations".
        - trigger_customizations_sfn_name                  = local.
            - this is the local file of the main folder where it will access the fsn name i.e., "aft-invoke-customizations".
        - aft_features_sfn_name                            = local.aft_features_sfn_name
            - this is the local file of the main folder where it will access the fsn name i.e.,"aft-account-provisioning-framework".".
        - aft_sns_topic_arn                                = module.aft_account_request_framework.aft_sns_topic_arn
            - this is accessing the module aft_account_request_framework and extracting the sns details sns.tf you can see in the outputs.tf for better understanding.
        - aft_failure_sns_topic_arn                        = module.aft_account_request_framework.aft_failure_sns_topic_arn
            - this is accessing the module aft_account_request_framework and extracting the sns details sns.tf you can see in the outputs.tf for better understanding.
        - aft_common_layer_arn                             = module.aft_lambda_layer.layer_version_arn
            - this is accessing the module aft_lambda_layer and extracting the layer version.
                - accessing first the outputs.tf then
                - then it will take us to the main.tf there is the code of "aws_lambda_layer_version"
        - aft_kms_key_arn                                  = module.aft_account_request_framework.aft_kms_key_arn
            - this is accessing the module aft-account-request-framework and extracting the aws_kms_key.
                - accessing first the outputs.tf then
                - then it will take us to the kms.tf there is code of "aft_kms_key_arn"
        - aft_vpc_private_subnets                          = module.aft_account_request_framework.aft_vpc_private_subnets
            - this is accessing the module aft-account-request-framework and extracting the aft_vpc_private_subnets.
                - accessing first the outputs.tf then
                - then it will take us to the vpc.tf there is code of "aft_vpc_private_subnet_01/02"
        - aft_vpc_default_sg                               = module.aft_account_request_framework.aft_vpc_default_sg
            - this is accessing the module aft-account-request-framework and extracting the aft_vpc_default_sg.
                - accessing first the outputs.tf then
                - then it will take us to the vpc.tf there is code of "aft_vpc_default_sg"
        - cloudwatch_log_group_retention                   = var.cloudwatch_log_group_retention
            - this is accessing variables.tf but didn't got the entry point for this
        - provisioning_framework_archive_path              = module.packaging.provisioning_framework_archive_path
            - it is accessing the aft-archives folder, where it will further access the output.tf first then data.tf

            data "archive_file" "provisioning_framework" {
                type        = "zip"
                source_dir  = "${path.module}/../../src/aft_lambda/aft_account_provisioning_framework"
                output_path = "${path.module}/../../src/aft_lambda/aft_account_provisioning_framework.zip"
            }

            - after data.tf it is accessing other path
        - provisioning_framework_archive_hash              = module.packaging.provisioning_framework_archive_hash
            - it is accessing the aft-archives folder, where it will further access the output.tf first then data.tf

            data "archive_file" "provisioning_framework" {
                type        = "zip"
                source_dir  = "${path.module}/../../src/aft_lambda/aft_account_provisioning_framework"
                output_path = "${path.module}/../../src/aft_lambda/aft_account_provisioning_framework.zip"
            }
    }

- Resources (Created)
    - <img width="203" alt="image" src="https://user-images.githubusercontent.com/57224583/196887828-58085aa3-51c6-4964-b61f-aa9ddc08c5ed.png">

    - IAM
        - Role-Policies
            - iam-aft-states.tpl
            - lambda-aft-account-provisioning-framework.tpl
        - Trust-Policies
            - lambda.tpl
            - states.tpl

    - States
        - aft_account_provisioning_framework.asl.json
    - data.tf
    - iam.tf
    - lambda.tf
        - It is creating functions for the following: 
            - VALIDATE REQUEST FUNCTION 
                - it is creating "validate request function" by taking inputs from different files like role, handler etc.
                - it is configuring a vpc as well by taking inputs from variables.tf file.
                - it is creating a resource of logging into cloudwatch for the particular validate_request function.

            -  GET ACCOUNT INFO FUNCTION 
                - it is creating "GET ACCOUNT INFO FUNCTION" by taking inputs from different files like role, handler etc.
                - it is configuring a vpc as well by taking inputs from variables.tf file.
                - it is creating a resource of logging into cloudwatch for the particular validate_request function.

            -  CREATE ROLE FUNCTION 

            -  TAG ACCOUNT FUNCTION 

            -  PERSIST METADATA FUNCTION 

            -  Account Metadata SSM Function 
    - locals.tf
        - values assigning names to expressions, so you can use these multiple times without repetition e.g. service_name = "forum"
    - outputs.tf
        - like return values for a Terraform module. Output value for consumption by another module or a human interacting via the UI.
    - states.tf
        - Local 
            - values assigning names to expressions, so you can use these multiple times without repetition e.g. service_name = "forum"

            - inside this there is a replacement_map for the references.

        - creating the below resource, to reference the above local parameter.

            resource "aws_sfn_state_machine" "aft_account_provisioning_framework_sfn" {
                name       = var.aft_account_provisioning_framework_sfn_name
                role_arn   = aws_iam_role.aft_states.arn
                definition = templatefile(local.state_machine_source, local.replacements_map)
            }

    - variables.tf
        - Input variable allowing users to customizate aspects of the configuration when used directly (e.g. via CLI, tfvars file or via environment variables), or as a module (via module arguments)
    - versions.tf
        - You can also set a version constraint for each provider defined in the required_providers block. The version attribute is optional, but we recommend using it to constrain the provider version so that terraform does not install a version of the provider that does not work with your configuration. If you do not specify a provider version, terraform will automatically download the most recent version during initialization.
