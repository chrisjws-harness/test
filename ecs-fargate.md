# ECS-Fargate Walkthrough 

## Create IAM Roles

1. Navigate to **IAM** and click **Create Role** 
2. Select AWS Service and click **Elastic Container Service**, then click **EC2 Role for Elastic Container Service**
3. Click next to **Permissions**
4. Click Next to **Tags**
5. Name this role **ECSDelegate** and click **Create Role **
6. Search for **ECSDelegate** role and click on it. 
7. Click **Attach Policies**
8. Click **Create Policy**
9. Navigate to the **JSON** tab
10. Paste the following JSON in the Policy Editor
	- `{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:DescribeRepositories",
                "ecs:ListClusters",
                "ecs:ListServices",
                "ecs:DescribeServices",
                "ecr:ListImages",
                "ecs:RegisterTaskDefinition",
                "ecs:CreateService",
                "ecs:ListTasks",
                "ecs:DescribeTasks",
                "ecs:DeleteService",
                "ecs:UpdateService",
                "ecs:DescribeContainerInstances",
                "ecs:DescribeTaskDefinition",
                "application-autoscaling:DescribeScalableTargets",
                "iam:ListRoles",
                "iam:PassRole"
            ],
            "Resource": "*"
        }
    ]
}`
`
11. Click Next
12. Click Next
13. Name the policy **HarnessECS**
14. Click **Create Policy** 
15. Search for **ECSDelegate** role and click into it.
16. Click Attach Policies
17. Search for and attach the **HarnessECS** policy we created earlier
18. Click Attach policies 
19. Search for and attach the folowing policies:
	- AmazonEC2ContainerServiceRole
20. Click on **Policies**
21. Click **Create Policy**
22. Click the JSON tab
23. Paste the following policies in: 
	- `{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "ec2:DescribeRegions",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:Get*",
                "s3:List*"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ecs:DescribeServices",
                "ecs:UpdateService",
                "cloudwatch:PutMetricAlarm",
                "cloudwatch:DescribeAlarms",
                "cloudwatch:DeleteAlarms"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Action": "ec2:*",
            "Effect": "Allow",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "elasticloadbalancing:*",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "cloudwatch:*",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "autoscaling:*",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:CreateOrUpdateTags",
                "autoscaling:DeleteTags",
                "autoscaling:DescribeTags"
            ],
            "Resource": "*"
        }
    ]
}`
24. Click Next
25. Click Next
26. Name this policy "Harness AWS Connector" and click **Create Policy**
27. Navigate back to **Roles** and search for the **ECSDelegate** Role
28. Attach the recently created "Harness AWS Connector" Policy to the **ECSDelegate** Role


## Cluster Creation 
Create a Cluster for the ECSDelegate

1. Navigate to **Elastic Container Service** 
2. Click **Create Cluster**
3. Select **Networking Only** and click next
4. Name this cluster **ECS-Delegate-Fargate**
5. Select **Create VPC**
6. Click **Create**
###Create the target cluster

1. Click Create Cluster
2. Name the cluster Target-Cluster
3. Use the same specs as above, however this cluster needs **2 registered container instances. **

## Download and install the Delegate

1. In Harness, navigate to Harness Delegates
2. Click Install Delegate
3. Select ECS Task Spec
4. Group name should be harness-ecs-delegates
5. Select Primary for the profile
6. Select Use AWS VPC Mode
7. Leave Hostname blank and ECS will defualt to the Docker Container ID as the hostname. 
8. Click Download
9. Untar the delegate file `tar -xzvf harness-delegate-ecs.tar.gz`
10. We'll need to change a few values in the task-spec. Open `service-spec-for-awsvpc-mode.json` and make the following changes: 
	- Set `requiresCompatibilities` to `FARGATE` instead of `EC2`.
	- Add `executionRoleArn` to the ECS task spec. This is needed by FARGATE launch types.
	- ex: `{
    "containerDefinitions": [
        {
        ...
        }
    ],
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "executionRoleArn": "arn:aws:iam::<your_account_id>:role/ecsTaskExecutionRole",
    "memory": "6144",
    "networkMode": "awsvpc",
    "cpu": "1024",
    "family": "harness-delegate-task-spec"
}`