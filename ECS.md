# ECS Walkthrough 
​
## Create IAM Roles
​
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
26. Name this policy **HarnessAWSConnector** and click **Create Policy**
27. Navigate back to **Roles** and search for the **ECSDelegate** Role
28. Attach the recently created "Harness AWS Connector" Policy to the **ECSDelegate** Role
​
## Cluster Creation 
Create a Cluster for the ECSDelegate
​
1. Navigate to **Elastic Container Service** 
2. Click **Create Cluster**
3. Select **EC2 Linux + Networking** and click next
4. Name this cluster **ECS-Delegate**
5. Select **On Demand Instance**
6. For **EC2 Instance Type** Select **m5.xlarge**
7. Select **1** Registered Container Instance
8. For **EC2 AMI ID** select **Amazon Linux 2 AMI**
9. Root EBS Volume Size (GiB) Leave at **30** 
10. Leave **Key Pair** empty
11. For Networking select **Create New VPC**
12. Leave **Create a new Security Group**
13. For Container Instance IAM Role select the **ECSDelegate** role we created in the previous step. 
14. Click **Create**
15. Click View Cluster
​
###Create the target cluster
​
1. Click Create Cluster
2. Name the cluster Target-Cluster
3. Use the same specs as above, however this cluster needs **2 registerd container instances. **
​
​
## Install the Delegate
​
1. In Harness, navigate to Install Delegate
2. For Delegate Group Name enter ECSDelegate
3. Leave Hostname Blank and click Download
4. Untar the delegate task spec
5. Run AWS Configure to ensure your spec is set correctly
6. Register the Task Definition using 
	`aws ecs register-task-definition --cli-input-json file://ecs-task-spec.json`
7. Review the completed task using `aws ecs list-task-definitions`
8. Create the ECS Service for the ECS-Delegate using the following command: `aws ecs create-service --service-name ecs-example --task-definition harness-delegate-task-spec --cluster ECS-Delegate --desired-count 1`
	(Make sure your cluster name matches the cluster created for the ECS Delegate in the previous step)
9. View the service using `aws ecs list-services --cluster ECS-Delegate`
