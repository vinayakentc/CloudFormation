

AWSTemplateFormatVersion: '2010-09-09'

Description: ThisThis template creates vpc with public and private subnets


# Parameters are used to to build flexible CloudFormation templates
Parameters:
  VpcCIDR:
    Default: 10.0.0.0/16
    Description: Please enter the IP range (CIDR notation) for this VPC
    Type: String

  PublicSubnetACIDR:
    Default: 10.0.0.0/24
    Description: Please enter the IP range (CIDR notation) for the public subnet 1
    Type: String

  PublicSubnetBCIDR:
    Default: 10.0.1.0/24
    Description: Please enter the IP range (CIDR notation) for the public subnet 2
    Type: String

  PrivateSubnetCCIDR:
    Default: 10.0.2.0/24
    Description: Please enter the IP range (CIDR notation) for the private subnet 1
    Type: String

  PrivateSubnetDCIDR:
    Default: 10.0.3.0/24
    Description: Please enter the IP range (CIDR notation) for the private subnet 2
    Type: String


  SSHLocation:
    AllowedPattern: '(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})'
    ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.
    Default: 0.0.0.0/0
    Description: The IP address range that can be used to access the web server using SSH.
    MaxLength: '18'
    MinLength: '9'
    Type: String

  VpcId: 
    Description: VPC id 
    Type: String
    Default: vpc-0fcc80c512efed028

  PublicSub1:
    Description: Subnet Id where instance will create 
    Type: String
    Default: subnet-04a6e662c8eacdcbe

  PubSubnet2:
    Description: Subnet Id where instance will create 
    Type: String
    Default: subnet-052d7277c767349a7

  PriSubnet3:
    Description: Subnet Id where instance will create 
    Type: String
    Default: subnet-0bb0da247fd3d05cb

  PrivateSubnet4:
    Description: Subnet Id where instance will create 
    Type: String
    Default: subnet-07fce02e8f702936a


  InstanceType:
    Description: WebServer EC2 instance type
    Type: String
    Default: t2.micro
    AllowedValues: [t2.micro]
    ConstraintDescription: must be a valid EC2 instance type.

  InstanceType2:
    Description: WebServer EC2 instance type
    Type: String
    Default: t2.micro
    AllowedValues: [t2.micro]
    ConstraintDescription: must be a valid EC2 instance type.


  Application:
    Description: Application Name
    Type: String
    AllowedPattern: "[A-Za-z0-9]+"
  
  Environment:
    AllowedValues: [dev,qa,venv]
    Default: venv
    Description: The name of the Environment
    Type: String

  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be the name of an existing EC2 KeyPair.

  LatestAmiId:
    Description: The ID of the Amazon machine image (AMI)
    Type: String

  LatestAmiId2:
    Description: The ID of the Amazon machine image (AMI)
    Type: String


  DatabaseInstanceIdentifier:
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9]*'
    ConstraintDescription: Must begin with a letter and contain only alphanumeric characters
    Default: database2
    Description: Instance identifier name
    MaxLength: 60
    MinLength: 1
    Type: String

  DatabaseName:
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9]*'
    ConstraintDescription: Must begin with a letter and contain only alphanumeric characters
    Default: ChatappDB
    Description: MySQL database name
    MaxLength: 64
    MinLength: 1
    Type: String

  DatabaseUser:
    AllowedPattern: '[a-zA-Z][a-zA-Z0-9]*'
    ConstraintDescription: Must begin with a letter and contain only alphanumeric characters
    Default: admin
    Description: Username for MySQL database access
    MaxLength: 16
    MinLength: 1
    NoEcho: true
    Type: String

  DatabasePassword:
    AllowedPattern: '[a-zA-Z0-9]*'
    ConstraintDescription: Must contain only alphanumeric characters
    Default: ganeshchavan
    Description: Password for MySQL database access
    MaxLength: 41
    MinLength: 8
    NoEcho: true
    Type: String

  DatabaseBackupRetentionPeriod:
    ConstraintDescription: Database backup retention period must be between 0 and 35 days
    Default: 0
    Description: The number of days for which automatic DB snapshots are retained
    MaxValue: 35
    MinValue: 0
    Type: Number

  DatabaseAllocatedStorage:
    ConstraintDescription: Must be between 5 and 1024Gb
    Default: 20
    Description: The size of the database (Gb)
    MaxValue: 65536
    MinValue: 5
    Type: Number

  DatabaseInstanceClass:
    AllowedValues:
      - db.t1.micro
      - db.t2.micro
      - db.m1.small
      - db.m1.medium
      - db.m1.large
    ConstraintDescription: Must select a valid database instance type
    Default: db.t2.micro
    Description: The database instance type
    Type: String

  MultiAZDatabase:
    AllowedValues:
      - true
      - false
    ConstraintDescription: Must be either true or false
    Default: false
    Description: Creates a Multi-AZ MySQL Amazon RDS database instance
    Type: String




Resources:
# Create VPC
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCIDR
      EnableDnsHostnames: true
      EnableDnsSupport: true
      InstanceTenancy: default
      Tags:
        - Key: Name
          Value: New_VPC

# Create Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: My_IGW

# Attach Internet Gateway to VPC
  InternetGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId: !Ref InternetGateway
      VpcId: !Ref VPC

# Create Public Subnet1
  PublicSub1:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      CidrBlock: !Ref PublicSubnetACIDR
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: Public Subnet A
      VpcId: !Ref VPC

# Create Public Subnet2
  PubSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [ 1, !GetAZs '' ]
      CidrBlock: !Ref PublicSubnetBCIDR
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: Public Subnet B
      VpcId: !Ref VPC

# Create Route Table
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      Tags:
        - Key: Name
          Value: Public Route Table
      VpcId: !Ref VPC

# Add a Public Route to the Route Table
  PublicRoute:
    Type: AWS::EC2::Route
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
      RouteTableId: !Ref PublicRouteTable

# Associate Public Subnet1 with Public Route Table
  PublicSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSub1

# Associate Public Subnet2 with Public Route Table
  PublicSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PubSubnet2

# Create Private Subnet1
  PriSubnet3:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [ 0, !GetAZs  '' ]
      CidrBlock: !Ref PrivateSubnetCCIDR
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: Private Subnet C
      VpcId: !Ref VPC

# Create Private Subnet2
  PrivateSubnet4:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select [ 1, !GetAZs '' ]
      CidrBlock: !Ref PrivateSubnetDCIDR
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: Private Subnet D
      VpcId: !Ref VPC

  # Allocate Elastic IP Address (EIP 1)
  NatGateway1EIP:
    Type: AWS::EC2::EIP
    Properties:
      Domain: VPC
      Tags:
        - Key: Name
          Value: EIP 1


# Create Nat Gateway 1 in Public Subnet 1    
  NatGateway1:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt NatGateway1EIP.AllocationId
      SubnetId: !Ref PublicSub1
      Tags:
        - Key: Name
          Value: Nat Gateway Public Subnet A


# Create Private Route Table 1 
  PrivateRouteTable1:
    Type: AWS::EC2::RouteTable
    Properties:
      Tags:
        - Key: Name
          Value: Private Route Table 1
      VpcId: !Ref VPC

# Add a route to point internet-bound traffic to Nat Gateway 1      
  PrivateRoute1:
    Type: AWS::EC2::Route
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGateway1
      RouteTableId: !Ref PrivateRouteTable1

# Associate Private Subnet 1 with Private Route Table 1
  PrivateSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PrivateRouteTable1
      SubnetId: !Ref PriSubnet3


# Create Private Route Table 2     
  PrivateRouteTable2:
    Type: AWS::EC2::RouteTable
    Properties:
      Tags:
        - Key: Name
          Value: Private Route Table 2
      VpcId: !Ref VPC

# Add a route to point internet-bound traffic to Nat Gateway 2         
  PrivateRoute2:
    Type: AWS::EC2::Route
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGateway1
      RouteTableId: !Ref PrivateRouteTable2

# Associate Private Subnet 2 with Private Route Table 2
  PrivateSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PrivateRouteTable2
      SubnetId: !Ref PrivateSubnet4





# Create Security Group for the Application Load Balancer 
  ALBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP access on port 80 and 8000
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 8000
          ToPort: 8000
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: ALB Security Group
      VpcId: !Ref VPC

# Create Security Group for the Base
  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: SSH Security Group
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: !Ref SSHLocation
      Tags:
        - Key: Name
          Value: SSH Security Group
      VpcId: !Ref VPC

# Create Security Group for the Web Server
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP access via port 80
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          CidrIp: 0.0.0.0/0
          ToPort: 80
        - IpProtocol: tcp
          FromPort: 8000
          CidrIp: 0.0.0.0/0
          ToPort: 8000
        - IpProtocol: tcp
          FromPort: 3306
          CidrIp: 0.0.0.0/0
          ToPort: 3306
        - IpProtocol: tcp
          FromPort: 22
          CidrIp: 0.0.0.0/0
          ToPort: 22
      Tags:
        - Key: Name
          Value: WebServer Security Group
      VpcId: !Ref VPC

  ApplicationLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name:  !Sub '${Application}-${Environment}-alb'
      Scheme: internet-facing
      Type: application
      IpAddressType: ipv4
      #KeyName: !Ref KeyName
      LoadBalancerAttributes:
        - Key: idle_timeout.timeout_seconds
          Value: 600
      SecurityGroups: 
        - !Ref ALBSecurityGroup
      Subnets:
        - !Ref PublicSub1
        - !Ref PubSubnet2

  ApplicationLoadBalancer2:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name:  !Sub '${Application}-${Environment}-alb2'
      Scheme: internal
      Type: application
      IpAddressType: ipv4
      #KeyName: !Ref KeyName
      LoadBalancerAttributes:
        - Key: idle_timeout.timeout_seconds
          Value: 600
      SecurityGroups: 
        - !Ref ALBSecurityGroup
      Subnets:
        - !Ref PriSubnet3
        - !Ref PrivateSubnet4

  ALBListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - TargetGroupArn: !Ref TargetGroup
          Type: forward
      LoadBalancerArn: !Ref ApplicationLoadBalancer
      Port: 80
      Protocol: HTTP

  ALBListener2:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - TargetGroupArn: !Ref TargetGroup2
          Type: forward
      LoadBalancerArn: !Ref ApplicationLoadBalancer2
      Port: 80
      Protocol: HTTP

  ALBListener3:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - TargetGroupArn: !Ref TargetGroup2
          Type: forward
      LoadBalancerArn: !Ref ApplicationLoadBalancer2
      Port: 8000
      Protocol: HTTP


  TargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      HealthCheckIntervalSeconds: 30
      HealthCheckProtocol: HTTP
      HealthyThresholdCount: 3
      HealthCheckPath: /
      Matcher:
        HttpCode: 200
      TargetGroupAttributes:
        - Key: deregistration_delay.timeout_seconds
          Value: 20
      VpcId: !Ref VPC
      Port: 80
      Protocol: HTTP
      UnhealthyThresholdCount: 3

  TargetGroup2:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      HealthCheckIntervalSeconds: 30
      HealthCheckProtocol: HTTP
      HealthyThresholdCount: 3
      HealthCheckPath: /
      Matcher:
        HttpCode: 200
      TargetGroupAttributes:
        - Key: deregistration_delay.timeout_seconds
          Value: 20
      VpcId: !Ref VPC
      Port: 8000
      Protocol: HTTP
      UnhealthyThresholdCount: 3

  LaunchConfig:
    Type: 'AWS::AutoScaling::LaunchConfiguration'
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      SecurityGroups:
        - !Ref WebServerSecurityGroup

  LaunchConfig2:
    Type: 'AWS::AutoScaling::LaunchConfiguration'
    Properties:
      ImageId: !Ref LatestAmiId2
      InstanceType: !Ref InstanceType2
      KeyName: !Ref KeyName
      SecurityGroups:
        - !Ref WebServerSecurityGroup

  AppServerASG:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      DesiredCapacity: 1
      HealthCheckGracePeriod: 300
      HealthCheckType: EC2
      LaunchConfigurationName: !Ref LaunchConfig
      MaxSize: 4
      MinSize: 1
      MetricsCollection:
        - Granularity: 1Minute
      Tags:
        - Key: Name
          PropagateAtLaunch: True
          Value: cloudapp1-dev-asg
      TargetGroupARNs:
        - !Ref TargetGroup
      TerminationPolicies:
        - OldestLaunchConfiguration
        - OldestInstance
        - Default
      VPCZoneIdentifier:
        - !Ref PublicSub1
        - !Ref PubSubnet2

  AppServerASG2:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      DesiredCapacity: 1
      HealthCheckGracePeriod: 300
      HealthCheckType: EC2
      LaunchConfigurationName: !Ref LaunchConfig2
      MaxSize: 4
      MinSize: 1
      MetricsCollection:
        - Granularity: 1Minute
      Tags:
        - Key: Name
          PropagateAtLaunch: True
          Value: cloudapp1-dev-asg2
      TargetGroupARNs:
        - !Ref TargetGroup2
      TerminationPolicies:
        - OldestLaunchConfiguration
        - OldestInstance
        - Default
      VPCZoneIdentifier:
        - !Ref PriSubnet3
        - !Ref PrivateSubnet4

  WebServerScaleUpPolicy:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AdjustmentType: ChangeInCapacity
      AutoScalingGroupName: !Ref 'AppServerASG'
      Cooldown: '300'
      ScalingAdjustment: 1

  WebServerScaleUpPolicy2:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AdjustmentType: ChangeInCapacity
      AutoScalingGroupName: !Ref 'AppServerASG2'
      Cooldown: '300'
      ScalingAdjustment: 1

  WebServerScaleDownPolicy:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AdjustmentType: ChangeInCapacity
      AutoScalingGroupName: !Ref 'AppServerASG'
      Cooldown: '60'
      ScalingAdjustment: -1

  WebServerScaleDownPolicy2:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AdjustmentType: ChangeInCapacity
      AutoScalingGroupName: !Ref 'AppServerASG2'
      Cooldown: '60'
      ScalingAdjustment: -1

  CPUAlarmHigh:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: Scale-up if CPU > 40% for 1 minutes
      MetricName: CPUUtilization
      Namespace: AWS/EC2
      Statistic: Average
      Period: 60
      EvaluationPeriods: 1
      Threshold: 10
      AlarmActions: [!Ref 'WebServerScaleUpPolicy']
      Dimensions:
      - Name: AutoScalingGroupName
        Value: !Ref 'AppServerASG'
      ComparisonOperator: GreaterThanThreshold

  CPUAlarmHigh2:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: Scale-up if CPU > 40% for 1 minutes
      MetricName: CPUUtilization
      Namespace: AWS/EC2
      Statistic: Average
      Period: 60
      EvaluationPeriods: 1
      Threshold: 10
      AlarmActions: [!Ref 'WebServerScaleUpPolicy2']
      Dimensions:
      - Name: AutoScalingGroupName
        Value: !Ref 'AppServerASG2'
      ComparisonOperator: GreaterThanThreshold

  CPUAlarmLow:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: Scale-down if CPU < 20% for 1 minutes
      MetricName: CPUUtilization
      Namespace: AWS/EC2
      Statistic: Average
      Period: 60
      EvaluationPeriods: 1
      Threshold: 5
      AlarmActions: [!Ref 'WebServerScaleDownPolicy']
      Dimensions:
      - Name: AutoScalingGroupName
        Value: !Ref 'AppServerASG'
      ComparisonOperator: LessThanThreshold

  CPUAlarmLow:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: Scale-down if CPU < 20% for 1 minutes
      MetricName: CPUUtilization
      Namespace: AWS/EC2
      Statistic: Average
      Period: 60
      EvaluationPeriods: 1
      Threshold: 5
      AlarmActions: [!Ref 'WebServerScaleDownPolicy2']
      Dimensions:
      - Name: AutoScalingGroupName
        Value: !Ref 'AppServerASG2'
      ComparisonOperator: LessThanThreshold



# Create Security Group for the DataBase
  DataBaseSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Open database for access
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref WebServerSecurityGroup
      Tags:
        - Key: Name
          Value: DataBase Security Group
      VpcId: !Ref VPC


  DatabaseSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: Subnet group for RDS database
      SubnetIds: 
        - !Ref PriSubnet3
        - !Ref PrivateSubnet4
      Tags:
        - Key: Name
          Value: database subnets

  DatabaseInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      AllocatedStorage: !Ref DatabaseAllocatedStorage
      AvailabilityZone: !Select [ 0, !GetAZs  '' ]
      BackupRetentionPeriod: !Ref DatabaseBackupRetentionPeriod
      DBInstanceClass: !Ref DatabaseInstanceClass
      DBInstanceIdentifier: !Ref DatabaseInstanceIdentifier
      DBName: !Ref DatabaseName
      DBSubnetGroupName: !Ref DatabaseSubnetGroup
      Engine: MySQL
      EngineVersion: 8.0.20
      MasterUsername: !Ref DatabaseUser
      MasterUserPassword: !Ref DatabasePassword
      MultiAZ: !Ref MultiAZDatabase







