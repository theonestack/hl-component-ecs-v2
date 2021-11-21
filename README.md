# ecs-v2 CfHighlander component
## Parameters

| Name | Use | Default | Global | Type | Allowed Values |
| ---- | --- | ------- | ------ | ---- | -------------- |
| EnvironmentName | Tagging | dev | true | string
| EnvironmentType | Tagging | development | true | string | ['development','production']
| VPCId | Security Groups | None | false | AWS::EC2::VPC::Id
| ContainerInsights | Whether to enable container insights | disabled | false | string | ['enabled','disabled']


If `fargate_only_cluster` is FALSE, the following parameters are accepted.

| Name | Use | Default | Global | Type | Allowed Values |
| ---- | --- | ------- | ------ | ---- | -------------- |
| KeyPair | keypair to use on EC2 instances | None | false | string
| Ami | AMI to use for EC2 instances | '/aws/service/ecs/optimized-ami/amazon-linux-2/recommended/image_id' | false | AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
| InstanceType | EC2 machine type | t3.small | false | string
| Spot | Whether to use spot instances | False | false | boolean
| Subnets | list of subnets | None | false | CommaDelimitedList
| AsgDesired | Desired instance count | 1 | false | int
| AsgMin | Desired minimum instance count | 1 | false | int
| AsgMax | Desired maximum instance count | 2 | false | int
| EnableScaling | Whether to enabling autoscaling | false | false | boolean
| EnableTargetTrackingScaling | Whether to enable target tracking scaling | false | false | boolean
## Outputs/Exports

| Name | Value | Exported |
| ---- | ----- | -------- |
| EcsCluster | ECS Cluster Name | true
| EcsClusterArn | ECS Cluster ARN | true
| EcsSecurityGroup | ECS Security Group Name | true
| AutoScalingGroupName | ASG Name | true

## Included Components

[lib-ec2](https://github.com/theonestack/hl-component-lib-ec2)
[lib-iam](https://github.com/theonestack/hl-component-lib-iam)

## Example Configuration
### Highlander
EC2 Cluster:
```
    Component name: 'ecsec2', template: 'ecs-v2', config: ecsec2 do
    parameter name: 'Ami', value: '/aws/service/ecs/optimized-ami/amazon-linux-2/recommended/image_id'
    parameter name: 'Subnets', value: cfout('vpcv2', 'ComputeSubnets')
    parameter name: 'InstanceType', value: cfmap('EnvironmentName', Ref('EnvironmentName'), 'InstanceType')
  end
```
Fargate cluster:
```
  Component name: 'ecsfargate', template: 'ecs-v2', config: ecsfargate do
    parameter name: 'Subnets', value: cfout('vpcv2', 'ComputeSubnets')
  end

```

### ECS-V2 Configuration
EC2 Cluster:
```
ecsec2:
  fargate_only_cluster: false
  cluster_name: ${EnvironmentName}-EC2-Cluster

  ecs_agent_config:
   ECS_AWSVPC_BLOCK_IMDS: true

  iam_policies:
   ecs-container-instance:
     action:
       - ecs:CreateCluster
       - ecs:DeregisterContainerInstance
       - ecs:DiscoverPollEndpoint
       - ecs:Poll
       - ecs:RegisterContainerInstance
       - ecs:StartTelemetrySession
       - ecs:Submit*
       - ecr:GetAuthorizationToken
       - ecr:BatchCheckLayerAvailability
       - ecr:GetDownloadUrlForLayer
       - ecr:BatchGetImage
       - logs:CreateLogStream
       - logs:PutLogEvents
       - cloudwatch:PutMetricData
   ecs-service-scheduler:
     action:
       - ec2:AuthorizeSecurityGroupIngress
       - ec2:Describe*
       - elasticloadbalancing:DeregisterInstancesFromLoadBalancer
       - elasticloadbalancing:DeregisterTargets
       - elasticloadbalancing:Describe*
       - elasticloadbalancing:RegisterInstancesWithLoadBalancer
       - elasticloadbalancing:RegisterTargets
   ecs-ssm:
     action:
       - ssmmessages:*
       - ssm:*
       - s3:GetEncryptionConfiguration

  dain_hook_iam_policies:
   ec2:
     action:
       - ec2:DescribeInstances
       - ec2:DescribeInstanceAttribute
       - ec2:DescribeInstanceStatus
       - ec2:DescribeHosts
   autoscaling:
     action:
       - autoscaling:CompleteLifecycleAction
     resource:
       - Fn::Sub: aws:aws:autoscaling:${AWS::Region}:${AWS::AccountId}:autoScalingGroup:*:autoScalingGroupName/${AutoScaleGroup}
   ecs1:
     action:
       - ecs:DescribeContainerInstances
       - ecs:DescribeTasks
   ecs2:
     action:
       - ecs:ListContainerInstances
       - ecs:SubmitContainerStateChange
       - ecs:SubmitTaskStateChange
     resource:
       - Fn::GetAtt: [EcsCluster,Arn]
   ecs3:
     action:
       - ecs:UpdateContainerInstancesState
       - ecs:ListTasks
     condition:
       ArnEquals:
         ecs:cluster:
           Fn::GetAtt: [EcsCluster,Arn]

  ecs_scaling_iam_policies:
   ecs1:
     action:
       - ecs:DescribeContainerInstances
       - ecs:DescribeTasks
   ecs2:
     action:
       - ecs:ListContainerInstances
     resource:
       - Fn::GetAtt: [EcsCluster,Arn]
   metrics:
     action:
       - cloudwatch:PutMetricData

  userdata: |
   cd /tmp
   sudo yum install -y https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm
   sudo systemctl enable amazon-ssm-agent
   sudo systemctl start amazon-ssm-agent
   sudo yum -y install amazon-cloudwatch-agent
   sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
   sudo yum install -y collectd
   sudo systemctl enable amazon-cloudwatch-agent
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c ssm:AmazonCloudWatch-linux
   sudo systemctl restart amazon-cloudwatch-agent
```

Fargate Cluster:
```
ecsfargate:
  fargate_only_cluster: true

  ecs_agent_config:
    ECS_AWSVPC_BLOCK_IMDS: true

  iam_policies:
    fargate_default_policy:
      action:
        - logs:GetLogEvents
      resource:
        - Fn::GetAtt:
            - LogGroup
            - Arn
```

## Cfhighlander Setup

install cfhighlander [gem](https://github.com/theonestack/cfhighlander)

```bash
gem install cfhighlander
```

or via docker

```bash
docker pull theonestack/cfhighlander
```
## Testing Components

Running the tests

```bash
cfhighlander cftest ecs-v2
```