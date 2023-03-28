# A Guide to deploying ECS Services on ip-saas-dev

     Important note:
        
        - This deployment is a step by step deployment, for easier debugging.
        - Resources created in the stacks to be deployed have exports which will be imported in the required resources i.e. !ImportValue ECSService.
        - The root files to each service has a url pointing to the s3 bucket containing the templates ensure that the url is correct in every deployment.
        - The templates relies heavily on string concatenation with two main parameters i.e. EnvironmentName and ServiceName. 
            * EnvironmentName - refers to dxhub
            * ServiceName - referes to the name of the service being aded i.e. neocrm-api, contactsvc

# 1. IAM 

    This is the first deployment to create in aws

    # Modifications

        # Root file

            TemplateURL: 

                This section reference the cloudformation template i.e. https://dxhub-infra-templates.s3.eu-central-1.amazonaws.com/iam/iam.yaml

    Then deploy the stack

# 2. Security

    # Security stack 

        In here you find two folders - internal-ecs-service and public-ecs-service

        If your service is publicly accessible use public-ecs-service and if your service is not accessible to the public use internal-ecs-service

        # Modifications

            # Root file

                TemplateURL: 

                    This section reference the cloudformation template i.e. https://dxhub-infra-templates.s3.eu-central-1.amazonaws.com/security/security.yaml

            # Security.yaml

                Change the port to suit your app port

    Then deploy the stack

# 3. ALB

    In here you find two folders - alb-internal and alb-internet-facing

    If your service is publicly accessible use alb-internet-facing and if your service is not accessible to the public use alb-internal

        # Alb internal Stack

            # Modifications

                # Target Group

                    Apps have different kind of health checks and Protocol Version - A need to understand what kind of healthcheck you need to implement is key

                    # Changes
                    
                        * Port to reflect the app port.

                        * Protocol Version
                    
                        * Health check path - Match this to reflect your health check path in your app.

                # ALB

                    Modify the Subnet section if your app resides in the Private Subnets or in the Public Subnets to use the required Subnet

        Then deploy the stack

        # Alb Internet-Facing Stack

            In here you find two folders - alb-internal and alb-internet-facing

            If your service is publicly accessible use alb-internet-facing and if your service is not accessible to the public use alb-internal

            # Modifications

                # Target Group

                    Apps have different kind of health checks and Protocol Version - A need to understand what kind of healthcheck you need to implement is key

                    # Changes
                    
                        * Port to reflect the app port.

                        * Protocol Version
                    
                        * Health check path - Match this to reflect your health check path in your app.

                # ALB

                    Modify the Subnet section if your app resides in the Private Subnets or in the Public Subnets to use the required Subnet

     Then deploy the stack

# 4. Parameters

    # Use this template to create parameters to be used in the ECS stacks.

    # Modifications

        # Export section

            The export section creates a concatenation on the export name that you can import using !ImportValue function then with the exact format of the concatenated string i.e.

                !ImportValue contactsvc-target-group-1-arn or !ImportValue contactsvc-prod-listener-arn

            # Important note to have is that this value is used in the ECS stacks in such a way i.e. !ImportValue contactsvc-ecs-task-role-arn


     Then deploy the stack

# 5. ECS

    # Resources to be imported here that were created in the previous stacks:

        - Iam Roles
        - Security Group
        - Subnets
        - Target groups

    # The resources to be imported above are already created from the previous deployments and some are part of the dxhub infra i.e. subnets

    # Method of import is of this formart !ImportValue PublicSubnet

    # Modifcations

        # TaskDefinition

            # Modifications in this section need to reflect the task-def.json on the codebase i.e. Secrets and Environment Variables

            # The Secrets and Environment Variables are Parameters and using them you need to modify the parameter section in the template and the root file template

        # ECS Service

            # Modifications in this section include - 

                - Subnets - Modify Subnets to the desired location of your service. ( The Dxhub infra allows you to choose between PublicSubnet 1-3 or PrivateSubnet 1-3)


     Then deploy the stack


# 6. Deployment

    # Resources to be imported here that were created in the previous stacks:

        - Parameters Section
        - Iam Roles - Codebuild, CodePipeline,
        - ECS Cluster
        - ECS Service
        - Target group - Target Group 1 and 2
        - Listeners - Prod and Test Listener

    # Modifications

        # Parameter Section

            - CodeStar Connection - if repo is not part of the JR Github Org
            - BranchName
            - S3 bucket - If you are using a new one replace the arn here

        # CodeBuild Project

            # Source Section - BuildSpec - This section refers to the buildspec-image.yml location in codebase

            # Confirm if you need the notify CodeBuildProject if not comment it out and also in codepipeline

        # CodePipeline

            # S3 bucket - reference a new output artifact bucket if need be
