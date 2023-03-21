# A Guide to deploying ECS Services on ip-saas-prod

    Important note:
        
        - This deployment is a step by step deployment, for easier debugging.
        - Infrastructutre Resources (VpcId, Public and Private Subnets, Security Groups) to be re-used are Referenced by Id or Name from AWS.
        - Resources created in the stacks to be deployed have exports which will be imported in the required resources i.e. !ImportValue ECSService.
        - The root files to each service has a url pointing to the s3 bucket containing the templates ensure that the url is correct in every deployment


# 1. IAM 

    This is the first deployment to create in aws

    # Modifications

        # Export section

            The export section creates a concatenation on the export name that you can import using !ImportValue function then with the exact format of the concatenated string i.e.

                !ImportValue contactsvc-target-group-1-arn or !ImportValue contactsvc-prod-listener-arn

            # Important note to have is that this value is used in the ECS stacks in such a way i.e. !ImportValue contactsvc-ecs-task-role-arn


            Then deploy the stack


# 2. Security

    # Security stack

        # Modifications

            # For service that are internet facing (accessible publicly)

                Check the comments in the file to know what ingress rule to use - to reflect the correct ingress rule to use. Also modify the logical ID number to reflect the number of rules i.e. (ServiceIngeressRule1). Modify the port section to reflect your app port.

            # For service that are not internet facing (not accessible publicly)

                Check the comments in the file to know what ingress rule to use - to reflect the correct ingress rule to use. Also modify the logical ID number to reflect the number of rules i.e. (ServiceIngeressRule1). Modify the port section to reflect your app port.

            # Export section

                The export section creates a concatenation on the export name that you can import using !ImportValue function then with the exact format of the concatenated string i.e.

                    !ImportValue contactsvc-target-group-1-arn or !ImportValue contactsvc-prod-listener-arn

                # Important note to have is that this value is used in the ECS stacks in such a way i.e. !ImportValue contactsvc-ecs-task-role-arn


    Then deploy the stack

# 3. ALB

    # Internal Stack

        Use this stack if your app is not publicy accessible.

        # Modifications

            # Target Group

                Port to reflect the app port.
                Health check path - Match this to reflect your health check path in your app.

            # ALB

                Modify the Subnet section if your app resides in the Private Subnets or in the Public Subnets.

            # Export section

                The export section creates a concatenation on the export name that you can import using !ImportValue function then with the exact format of the concatenated string i.e.

                    !ImportValue contactsvc-target-group-1-arn or !ImportValue contactsvc-prod-listener-arn

                # Important note to have is that this value is used in the ECS stacks in such a way i.e. !ImportValue contactsvc-ecs-task-role-arn

        Then deploy the stack

    # Internet-Facing Stack

        Use this stack if your app is publicy accessible.

        # Modifications

            # Target Group

                Port to reflect the app port.
                Health check path - Match this to reflect your health check path in your app.

            # ALB

                Modify the Subnet section if your app resides in the Private Subnets or in the Public Subnets.


            # Export section

                The export section creates a concatenation on the export name that you can import using !ImportValue function then with the exact format of the concatenated string i.e.

                    !ImportValue contactsvc-target-group-1-arn or !ImportValue contactsvc-prod-listener-arn

                # Important note to have is that this value is used in the ECS stacks in such a way i.e. !ImportValue contactsvc-ecs-task-role-arn

     Then deploy the stack

# 4. Parameters

    # Use this template to create parameters to be used in the ECS stacks.

    # Modifications

       # Export section

            The export section creates a concatenation on the export name that you can import using !ImportValue function then with the exact format of the concatenated string i.e.

                !ImportValue contactsvc-target-group-1-arn or !ImportValue contactsvc-prod-listener-arn

            # Important note to have is that this value is used in the ECS stacks in such a way i.e. !ImportValue contactsvc-ecs-task-role-arn


     Then deploy the stack

# 4. ECS

    # Resources to be imported here that were created in the previous stacks:

        - Iam Roles
        - Security Group
        - Subnets - Modify Subnets to the desired location of your service. ( The Dxhub infra allows you to choose between PublicSubnet 1-3 or PrivateSubnet 1-3)
        - Target groups 

    # The resources to be imported above are already created from the previous deployments and some are part of the dxhub infr i.e. subnets

    # Method of import is of this formart !ImportValue PublicSubnet

    # Modifcations

    # TaskDefinition

        # Modifications in this section need to reflect the task-def.json on the codebase.

        # You will need to create Parameters in the parameters section for the Environment Variables and the Secrets Variables too.

        # ImportValues here are values from iam -

            - ecs-task-execution-role-arn i.e. !ImportValue ECSTaskExecutionRoleArn
            - task-role-arn i.e. !ImportValue ECSTaskRoleArn

    # CloudWatchLogs

        # This section we have a static value in the LogGroupName because the service being created belongs to the dxhub cluster

    # ECS Service

         # Modifications in this section include !ImportValues on the following -

            - Cluster
            - TargetGroup1Arn
            - TargetGroup1Arn
            - SecurityGroup
            - Subnets

    # Export section

        The export section creates a concatenation on the export name that you can import using !ImportValue function then with the exact format of the concatenated string i.e.

            !ImportValue contactsvc-target-group-1-arn or !ImportValue contactsvc-prod-listener-arn

        # Important note to have is that this value is used in the ECS stacks in such a way i.e. !ImportValue contactsvc-ecs-task-role-arn

     Then deploy the stack


# 5. Deployment

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

            # Confirm your ServiceRole ImportValue

            # Confirm if you need the notify CodeBuildProject if not comment it out and also in codepipeline

        # CodeDeploy DeploymentGroup

            # Modifications

                # ImportValues

                    - IAM - CodeDeploy Service Role
                    - ECS ClusterName
                    - ECS ServiceName
                    - ListernerArns - Prod and Test
                    - TargetGroup - 1 and 2
                    - IAM - CodeDeploy

        # CodePipeline

            # Modifications

                # ImportValues

                    - IAM - CodePipeline Role
