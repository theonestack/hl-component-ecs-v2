# ecs-v2 CfHighlander component

![cftest](https://github.com/theonestack/hl-component-ecs-v2/actions/workflows/rspec.yaml/badge.svg)


## Parameters

| Name | Use | Default | Global | Type | Allowed Values |
| ---- | --- | ------- | ------ | ---- | -------------- |
| EnvironmentName | Tagging | dev | true | string
| EnvironmentType | Tagging | development | true | string | ['development','production']
| VPCId | Security Groups | None | false | AWS::EC2::VPC::Id

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
`  Component name: 'ecsv2', template: 'ecs-v2' do
    parameter name: 'Ami', value: '/aws/service/ecs/optimized-ami/amazon-linux-2/recommended/image_id'
    parameter name: 'Subnets', value: cfout('vpcv2', 'ComputeSubnets')
  end`
