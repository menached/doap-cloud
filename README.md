doap-cloud
==========

This is the script that creates the Doap Virtual Private Cloud

Begin Script:
==========

{
  "AWSTemplateFormatVersion" : "2010-09-09",

  "Description" : "Doap (Deploy on any platform) is a multi-tier web app framework in a VPC with multiple subnets. The first subnet is public and contains and internet facing load balancer, a NAT device for internet access from the private subnet and a development (bastion) host to allow SSH access to the hosts in the private subnet. The second subnet is private and contains a Frontend fleet of EC2 instances, an internal load balancer and a Backend fleet of EC2 instances.",

  "Parameters" : {

    "KeyName" : {
  "Default" : "doap-norcal",
      "Description" : "The Doap KeyPair to enable SSH access to the instances",
      "Type" : "String",
      "MinLength": "1",
      "MaxLength": "64",
      "AllowedPattern" : "[-_ a-zA-Z0-9]*",
      "ConstraintDescription" : "can contain only alphanumeric characters, spaces, dashes and underscores."
    },

    "SSHFrom" : {
      "Description" : "Lockdown SSH access to the Development(bastion) host",
      "Type" : "String",
      "MinLength": "9",
      "MaxLength": "18",
      "Default" : "0.0.0.0/0",
      "AllowedPattern" : "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})",
      "ConstraintDescription" : "must be a valid CIDR range of the form x.x.x.x/x."
    },

   "HostedZone" : {
      "Type" : "String",
      "Default" : "doap.com",
      "Description" : "The DNS name of an existing Amazon Route 53 hosted zone"
    },

    "FrontendInstanceType" : {
      "Description" : "Frontend Server EC2 instance type",
      "Type" : "String",
      "Default" : "t1.micro",
      "AllowedValues" : [ "t1.micro","m1.small","m1.medium","m1.large","m1.xlarge","m2.xlarge","m2.2xlarge","m2.4xlarge","c1.medium","c1.xlarge","cc1.4xlarge","cc2.8xlarge","cg1.4xlarge"],
      "ConstraintDescription" : "must be a valid EC2 instance type."
    },

    "FrontendSize" : {
      "Description" : "Number of EC2 instances to launch for the Frontend server",
      "Type" : "Number",
      "Default" : "1"
    },

    "BackendInstanceType" : {
      "Description" : "Backend Server EC2 instance type",
      "Type" : "String",
      "Default" : "t1.micro",
      "AllowedValues" : [ "t1.micro","m1.small","m1.medium","m1.large","m1.xlarge","m2.xlarge","m2.2xlarge","m2.4xlarge","c1.medium","c1.xlarge","cc1.4xlarge","cc2.8xlarge","cg1.4xlarge"],
      "ConstraintDescription" : "must be a valid EC2 instance type."
    },

    "BackendSize" : {
      "Description" : "Number of EC2 instances to launch for the backend server",
      "Type" : "Number",
      "Default" : "1"
    },

    "BastionInstanceType" : {
      "Description" : "Bastion Host EC2 instance type",
      "Type" : "String",
      "Default" : "t1.micro",
      "AllowedValues" : [ "t1.micro","m1.small","m1.medium","m1.large","m1.xlarge","m2.xlarge","m2.2xlarge","m2.4xlarge","c1.medium","c1.xlarge","cc1.4xlarge","cc2.8xlarge","cg1.4xlarge"],
      "ConstraintDescription" : "must be a valid EC2 instance type."
    },

    "NATInstanceType" : {
      "Description" : "NET Device EC2 instance type",
      "Type" : "String",
      "Default" : "t1.micro",
      "AllowedValues" : [ "t1.micro","m1.small","m1.medium","m1.large","m1.xlarge","m2.xlarge","m2.2xlarge","m2.4xlarge","c1.medium","c1.xlarge","cc1.4xlarge","cc2.8xlarge","cg1.4xlarge"],
      "ConstraintDescription" : "must be a valid EC2 instance type."
    }
  },

  "Mappings" : {
    "AWSNATAMI" : {
      "us-east-1"      : { "AMI" : "ami-c6699baf" },
      "us-west-2"      : { "AMI" : "ami-52ff7262" },
      "us-west-1"      : { "AMI" : "ami-3bcc9e7e" },
      "eu-west-1"      : { "AMI" : "ami-0b5b6c7f" },
      "ap-southeast-1" : { "AMI" : "ami-02eb9350" },
      "ap-northeast-1" : { "AMI" : "ami-14d86d15" },
      "sa-east-1"      : { "AMI" : "ami-0439e619" }
    },

    "AWSInstanceType2Arch" : {
      "t1.micro"    : { "Arch" : "64" },
      "m1.small"    : { "Arch" : "64" },
      "m1.medium"   : { "Arch" : "64" },
      "m1.large"    : { "Arch" : "64" },
      "m1.xlarge"   : { "Arch" : "64" },
      "m2.xlarge"   : { "Arch" : "64" },
      "m2.2xlarge"  : { "Arch" : "64" },
      "m2.4xlarge"  : { "Arch" : "64" },
      "c1.medium"   : { "Arch" : "64" },
      "c1.xlarge"   : { "Arch" : "64" },
      "cc1.4xlarge" : { "Arch" : "64Cluster" },
      "cc2.8xlarge" : { "Arch" : "64Cluster" },
      "cg1.4xlarge" : { "Arch" : "64GPU" }
    },

    "AWSRegionArch2AMI" : {
      "us-east-1"      : { "32" : "ami-a0cd60c9", "64" : "ami-aecd60c7", "64Cluster" : "ami-a8cd60c1",      "64GPU" : "ami-eccf6285" },
      "us-west-2"      : { "32" : "ami-46da5576", "64" : "ami-48da5578", "64Cluster" : "NOT_YET_SUPPORTED", "64GPU" : "NOT_YET_SUPPORTED" },
      "us-west-1"      : { "32" : "ami-7d4c6938", "64" : "ami-734c6936", "64Cluster" : "NOT_YET_SUPPORTED", "64GPU" : "NOT_YET_SUPPORTED" },
      "eu-west-1"      : { "32" : "ami-61555115", "64" : "ami-6d555119", "64Cluster" : "ami-67555113",      "64GPU" : "NOT_YET_SUPPORTED" },
      "ap-southeast-1" : { "32" : "ami-220b4a70", "64" : "ami-3c0b4a6e", "64Cluster" : "NOT_YET_SUPPORTED", "64GPU" : "NOT_YET_SUPPORTED" },
      "ap-northeast-1" : { "32" : "ami-2a19aa2b", "64" : "ami-2819aa29", "64Cluster" : "NOT_YET_SUPPORTED", "64GPU" : "NOT_YET_SUPPORTED" },
      "sa-east-1"      : { "32" : "ami-f836e8e5", "64" : "ami-fe36e8e3", "64Cluster" : "NOT_YET_SUPPORTED", "64GPU" : "NOT_YET_SUPPORTED" }
    },

    "SubnetConfig" : {
      "VPC"     : { "CIDR" : "10.0.0.0/16" },
      "Public"  : { "CIDR" : "10.0.0.0/24" },
      "Private" : { "CIDR" : "10.0.1.0/24" }
    }
  },

  "Resources" : {

    "CfnUser" : {
      "Type" : "AWS::IAM::User",
      "Properties" : {
        "Path": "/",
        "Policies": [{
          "PolicyName": "root",
          "PolicyDocument": { "Statement":[{
            "Effect":"Allow",
            "Action":"cloudformation:DescribeStackResource",
            "Resource":"*"
          }]}
        }]
      }
    },

    "HostKeys" : {
      "Type" : "AWS::IAM::AccessKey",
      "Properties" : {
        "UserName" : { "Ref": "CfnUser" }
      }
    },

    "VPC" : {
      "Type" : "AWS::EC2::VPC",
      "Properties" : {
        "CidrBlock" : { "Fn::FindInMap" : [ "SubnetConfig", "VPC", "CIDR" ]},
        "Tags" : [
          { "Key" : "Application", "Value" : { "Ref" : "AWS::StackName" } },
          { "Key" : "Network", "Value" : "Public" }
        ]
      }
    },

    "PublicSubnet" : {
      "Type" : "AWS::EC2::Subnet",
      "Properties" : {
        "VpcId" : { "Ref" : "VPC" },
        "CidrBlock" : { "Fn::FindInMap" : [ "SubnetConfig", "Public", "CIDR" ]},
        "Tags" : [
          { "Key" : "Application", "Value" : { "Ref" : "AWS::StackName" } },
          { "Key" : "Network", "Value" : "Public" }
        ]
      }
    },

    "InternetGateway" : {
      "Type" : "AWS::EC2::InternetGateway",
      "Properties" : {
        "Tags" : [
          { "Key" : "Application", "Value" : { "Ref" : "AWS::StackName" } },
          { "Key" : "Network", "Value" : "Public" }
        ]
      }
    },

    "GatewayToInternet" : {
       "Type" : "AWS::EC2::VPCGatewayAttachment",
       "Properties" : {
         "VpcId" : { "Ref" : "VPC" },
         "InternetGatewayId" : { "Ref" : "InternetGateway" }
       }
    },

    "PublicRouteTable" : {
      "Type" : "AWS::EC2::RouteTable",
      "Properties" : {
        "VpcId" : { "Ref" : "VPC" },
        "Tags" : [
          { "Key" : "Application", "Value" : { "Ref" : "AWS::StackName" } },
          { "Key" : "Network", "Value" : "Public" }
        ]
      }
    },

    "PublicRoute" : {
      "Type" : "AWS::EC2::Route",
      "Properties" : {
        "RouteTableId" : { "Ref" : "PublicRouteTable" },
        "DestinationCidrBlock" : "0.0.0.0/0",
        "GatewayId" : { "Ref" : "InternetGateway" }
      }
    },

    "PublicSubnetRouteTableAssociation" : {
      "Type" : "AWS::EC2::SubnetRouteTableAssociation",
      "Properties" : {
        "SubnetId" : { "Ref" : "PublicSubnet" },
        "RouteTableId" : { "Ref" : "PublicRouteTable" }
      }
    },

    "PublicNetworkAcl" : {
      "Type" : "AWS::EC2::NetworkAcl",
      "Properties" : {
        "VpcId" : { "Ref" : "VPC" },
        "Tags" : [
          { "Key" : "Application", "Value" : { "Ref" : "AWS::StackName" } },
          { "Key" : "Network", "Value" : "Public" }
        ]
      }
    },

    "InboundHTTPPublicNetworkAclEntry" : {
      "Type" : "AWS::EC2::NetworkAclEntry",
      "Properties" : {
        "NetworkAclId" : { "Ref" : "PublicNetworkAcl" },
        "RuleNumber" : "100",
        "Protocol" : "6",
        "RuleAction" : "allow",
        "Egress" : "false",
        "CidrBlock" : "0.0.0.0/0",
        "PortRange" : { "From" : "80", "To" : "80" }
      }
    },

    "InboundHTTPSPublicNetworkAclEntry" : {
      "Type" : "AWS::EC2::NetworkAclEntry",
      "Properties" : {
        "NetworkAclId" : { "Ref" : "PublicNetworkAcl" },
        "RuleNumber" : "101",
        "Protocol" : "6",
        "RuleAction" : "allow",
        "Egress" : "false",
        "CidrBlock" : "0.0.0.0/0",
        "PortRange" : { "From" : "443", "To" : "443" }
      }
    },

    "InboundSSHPublicNetworkAclEntry" : {
      "Type" : "AWS::EC2::NetworkAclEntry",
      "Properties" : {
        "NetworkAclId" : { "Ref" : "PublicNetworkAcl" },
        "RuleNumber" : "102",
        "Protocol" : "6",
        "RuleAction" : "allow",
        "Egress" : "false",
        "CidrBlock" : { "Ref" : "SSHFrom" },
        "PortRange" : { "From" : "22", "To" : "22" }
      }
    },

    "InboundEmphemeralPublicNetworkAclEntry" : {
      "Type" : "AWS::EC2::NetworkAclEntry",
      "Properties" : {
        "NetworkAclId" : { "Ref" : "PublicNetworkAcl" },
        "RuleNumber" : "103",
        "Protocol" : "6",
        "RuleAction" : "allow",
        "Egress" : "false",
        "CidrBlock" : "0.0.0.0/0",
        "PortRange" : { "From" : "1024", "To" : "65535" }
      }
    },

    "OutboundPublicNetworkAclEntry" : {
      "Type" : "AWS::EC2::NetworkAclEntry",
      "Properties" : {
        "NetworkAclId" : { "Ref" : "PublicNetworkAcl" },
        "RuleNumber" : "100",
        "Protocol" : "6",
        "RuleAction" : "allow",
        "Egress" : "true",
        "CidrBlock" : "0.0.0.0/0",
        "PortRange" : { "From" : "0", "To" : "65535" }
      }
    },

    "PublicSubnetNetworkAclAssociation" : {
      "Type" : "AWS::EC2::SubnetNetworkAclAssociation",
      "Properties" : {
        "SubnetId" : { "Ref" : "PublicSubnet" },
        "NetworkAclId" : { "Ref" : "PublicNetworkAcl" }
      }
    },

    "PrivateSubnet" : {
      "Type" : "AWS::EC2::Subnet",
      "Properties" : {
        "VpcId" : { "Ref" : "VPC" },
        "CidrBlock" : { "Fn::FindInMap" : [ "SubnetConfig", "Private", "CIDR" ]},
        "Tags" : [
          { "Key" : "Application", "Value" : { "Ref" : "AWS::StackName" } },
          { "Key" : "Network", "Value" : "Private" }
        ]
      }
    },

    "PrivateRouteTable" : {
      "Type" : "AWS::EC2::RouteTable",
      "Properties" : {
        "VpcId" : { "Ref" : "VPC" },
        "Tags" : [
          { "Key" : "Application", "Value" : { "Ref" : "AWS::StackName" } },
          { "Key" : "Network", "Value" : "Private" }
        ]
      }
    },

    "PrivateSubnetRouteTableAssociation" : {
      "Type" : "AWS::EC2::SubnetRouteTableAssociation",
      "Properties" : {
        "SubnetId" : { "Ref" : "PrivateSubnet" },
        "RouteTableId" : { "Ref" : "PrivateRouteTable" }
      }
    },

    "PrivateRoute" : {
      "Type" : "AWS::EC2::Route",
      "Properties" : {
        "RouteTableId" : { "Ref" : "PrivateRouteTable" },
        "DestinationCidrBlock" : "0.0.0.0/0",
        "InstanceId" : { "Ref" : "NATDevice" }
      }
    },

    "PrivateNetworkAcl" : {
      "Type" : "AWS::EC2::NetworkAcl",
      "Properties" : {
        "VpcId" : { "Ref" : "VPC" },
        "Tags" : [
          { "Key" : "Application", "Value" : { "Ref" : "AWS::StackName" } },
          { "Key" : "Network", "Value" : "Private" }
        ]
      }
    },

    "InboundPrivateNetworkAclEntry" : {
      "Type" : "AWS::EC2::NetworkAclEntry",
      "Properties" : {
        "NetworkAclId" : { "Ref" : "PrivateNetworkAcl" },
        "RuleNumber" : "100",
        "Protocol" : "6",
        "RuleAction" : "allow",
        "Egress" : "false",
        "CidrBlock" : "0.0.0.0/0",
        "PortRange" : { "From" : "0", "To" : "65535" }
      }
    },

    "OutBoundPrivateNetworkAclEntry" : {
      "Type" : "AWS::EC2::NetworkAclEntry",
      "Properties" : {
        "NetworkAclId" : { "Ref" : "PrivateNetworkAcl" },
        "RuleNumber" : "100",
        "Protocol" : "6",
        "RuleAction" : "allow",
        "Egress" : "true",
        "CidrBlock" : "0.0.0.0/0",
        "PortRange" : { "From" : "0", "To" : "65535" }
      }
    },

    "PrivateSubnetNetworkAclAssociation" : {
      "Type" : "AWS::EC2::SubnetNetworkAclAssociation",
      "Properties" : {
        "SubnetId" : { "Ref" : "PrivateSubnet" },
        "NetworkAclId" : { "Ref" : "PrivateNetworkAcl" }
      }
    },

    "NATIPAddress" : {
      "Type" : "AWS::EC2::EIP",
      "Properties" : {
        "Domain" : "vpc",
        "InstanceId" : { "Ref" : "NATDevice" }
      }
    },

    "NATDevice" : {
      "Type" : "AWS::EC2::Instance",
      "Properties" : {
        "InstanceType" : { "Ref" : "NATInstanceType" },
        "KeyName" : { "Ref" : "KeyName" },
        "SubnetId" : { "Ref" : "PublicSubnet" },
        "SourceDestCheck" : "false",
        "ImageId" : { "Fn::FindInMap" : [ "AWSNATAMI", { "Ref" : "AWS::Region" }, "AMI" ]},
        "SecurityGroupIds" : [{ "Ref" : "NATSecurityGroup" }]
      }
    },

    "NATSecurityGroup" : {
      "Type" : "AWS::EC2::SecurityGroup",
      "Properties" : {
        "GroupDescription" : "Enable internal access to the NAT device",
        "VpcId" : { "Ref" : "VPC" },
        "SecurityGroupIngress" : [
           { "IpProtocol" : "tcp", "FromPort" : "80",  "ToPort" : "80",  "CidrIp" : "0.0.0.0/0" } ,
           { "IpProtocol" : "tcp", "FromPort" : "443", "ToPort" : "443", "CidrIp" : "0.0.0.0/0" } ,
           { "IpProtocol" : "tcp", "FromPort" : "22",  "ToPort" : "22",  "CidrIp" : { "Ref" : "SSHFrom" }} ],
        "SecurityGroupEgress" : [
           { "IpProtocol" : "tcp", "FromPort" : "80",  "ToPort" : "80",  "CidrIp" : "0.0.0.0/0" } ,
           { "IpProtocol" : "tcp", "FromPort" : "443", "ToPort" : "443", "CidrIp" : "0.0.0.0/0" } ]
      }
    },

 "myNatDNSRecord" : {
      "Type" : "AWS::Route53::RecordSet",
      "Properties" : {
        "HostedZoneName" : { "Fn::Join" : [ "", [{"Ref" : "HostedZone"}, "." ]]},
        "Comment" : "DNS name for my Nat instance.",
        "Name" : { "Fn::Join" : [ "", ["nat", ".", {"Ref" : "HostedZone"} ,"."]]},
        "Type" : "A",
        "TTL" : "900",
        "ResourceRecords" : [ { "Ref" : "NATIPAddress"} ]
      }
    },


    "BastionIPAddress" : {
      "Type" : "AWS::EC2::EIP",
      "Properties" : {
        "Domain" : "vpc",
        "InstanceId" : { "Ref" : "BastionHost" }
      }
    },

    "BastionHost" : {
      "Type" : "AWS::EC2::Instance",
      "Properties" : {
        "InstanceType" : { "Ref" : "BastionInstanceType" },
        "KeyName"  : { "Ref" : "KeyName" },
        "SubnetId" : { "Ref" : "PublicSubnet" },
        "ImageId"  : { "Fn::FindInMap" : [ "AWSRegionArch2AMI", { "Ref" : "AWS::Region" }, { "Fn::FindInMap" : [ "AWSInstanceType2Arch", { "Ref" : "BastionInstanceType" }, "Arch" ] } ] },
        "SecurityGroupIds" : [{ "Ref" : "BastionSecurityGroup" }]
      }
    },

    "BastionSecurityGroup" : {
      "Type" : "AWS::EC2::SecurityGroup",
      "Properties" : {
        "GroupDescription" : "Enable access to the Bastion host",
        "VpcId" : { "Ref" : "VPC" },
        "SecurityGroupIngress" : [ { "IpProtocol" : "tcp", "FromPort" : "22",  "ToPort" : "22",  "CidrIp" : { "Ref" : "SSHFrom" }} ],
        "SecurityGroupEgress"  : [ { "IpProtocol" : "tcp", "FromPort" : "22",  "ToPort" : "22",  "CidrIp" : { "Fn::FindInMap" : [ "SubnetConfig", "Private", "CIDR" ]}}]
      }
    },

 "myBastionDNSRecord" : {
      "Type" : "AWS::Route53::RecordSet",
      "Properties" : {
        "HostedZoneName" : { "Fn::Join" : [ "", [{"Ref" : "HostedZone"}, "." ]]},
        "Comment" : "DNS name for my instance.",
        "Name" : { "Fn::Join" : [ "", ["ssh", ".", {"Ref" : "HostedZone"} ,"."]]},
        "Type" : "A",
        "TTL" : "900",
        "ResourceRecords" : [ { "Fn::GetAtt" : [ "BastionHost", "PublicIp" ] } ]
      }
    },

    "PublicElasticLoadBalancer" : {
      "Type" : "AWS::ElasticLoadBalancing::LoadBalancer",
      "Properties" : {
        "SecurityGroups" : [ { "Ref" : "PublicLoadBalancerSecurityGroup" } ],
        "Subnets" : [ { "Ref" : "PublicSubnet" } ],
        "Listeners" : [ { "LoadBalancerPort" : "80", "InstancePort" : "80", "Protocol" : "HTTP" } ],
        "HealthCheck" : {
          "Target" : "HTTP:80/index.html",
          "HealthyThreshold" : "3",
          "UnhealthyThreshold" : "5",
          "Interval" : "90",
          "Timeout" : "60"
        }
      }
    },

    "PublicLoadBalancerSecurityGroup" : {
      "Type" : "AWS::EC2::SecurityGroup",
      "Properties" : {
        "GroupDescription" : "Public ELB Security Group with HTTP access on port 80 from the internet",
        "VpcId" : { "Ref" : "VPC" },
        "SecurityGroupIngress" : [ { "IpProtocol" : "tcp", "FromPort" : "80", "ToPort" : "80", "CidrIp" : "0.0.0.0/0" } ],
        "SecurityGroupEgress" : [ { "IpProtocol" : "tcp", "FromPort" : "80", "ToPort" : "80", "CidrIp" : "0.0.0.0/0" } ]
      }
    },

    "myLBDNSRecord" : {
      "Type" : "AWS::Route53::RecordSet",
      "Properties" : {
        "HostedZoneName" : { "Fn::Join" : [ "", [{"Ref" : "PublicElasticLoadBalancer"}, "." ]]},
        "Comment" : "CNAME redirect to lb.doap.com.",
        "Name" : { "Fn::GetAtt" : [ "PublicElasticLoadBalancer", "DNSName" ]},
        "Type" : "CNAME",
        "TTL" : "900",
        "ResourceRecords" : ["lb.doap.com"]
      }
    },

    "FrontendFleet" : {
      "Type" : "AWS::AutoScaling::AutoScalingGroup",
      "Properties" : {
        "AvailabilityZones" : [{ "Fn::GetAtt" : [ "PrivateSubnet", "AvailabilityZone" ] }],
        "VPCZoneIdentifier" : [{ "Ref" : "PrivateSubnet" }],
        "LaunchConfigurationName" : { "Ref" : "FrontendServerLaunchConfig"  },
        "MinSize" : "1",
        "MaxSize" : "10",
        "DesiredCapacity" : { "Ref" : "FrontendSize" },
        "LoadBalancerNames" : [ { "Ref" : "PublicElasticLoadBalancer" } ],
        "Tags" : [ { "Key" : "Network", "Value" : "Public", "PropagateAtLaunch" : "true" } ]
      }
    },

    "FrontendServerLaunchConfig"  : {
      "Type" : "AWS::AutoScaling::LaunchConfiguration",
      "Metadata" : {
        "Comment1" : "Configure the FrontendServer to forward /backend requests to the backend servers",

        "AWS::CloudFormation::Init" : {
          "config" : {
            "packages" : {
              "yum" : {
                "mailx"            : [],
                "lynx"             : [],
                "mysql"            : [],
                "mysql-server"     : [],
                "mysql-libs"       : [],
                "httpd"            : [],
                "php"              : [],
                "php-mysql"        : [],
                "php-common"       : [],
                "php-cli"          : [],
                "php-gd"           : [],
                "php-xml"          : [],
                "php-mbstring"     : [],
                "mysql-devel"      : [],
                "php-pecl-memcache" : [],
                "ImageMagick"      : [],
                "memcached"        : []              }
            },

            "files" : {
              "/var/www/html/index.html" : {
                "content" : { "Fn::Join" : ["\n", [
                  "<img src=\"https://s3.amazonaws.com/cloudformation-examples/cloudformation_graphic.png\" alt=\"AWS CloudFormation Logo\"/>",
                  "<h1>Congratulations, the soap ecosystem has been launched.</h1>",
                  "<p>This is a multi-tier web application launched in an Amazon Virtual Private Cloud (Amazon VPC) with multiple subnets. The first subnet is public and contains and internet facing load balancer, a NAT device for internet access from the private subnet and a bastion host to allow SSH access to the hosts in the private subnet. The second subnet is private and contains a Frontend fleet of EC2 instances, an internal load balancer and a Backend fleet of EC2 instances.",
                  "<p>To serve a web page from the backend service, click <a href=\"/backend\">here</a>.</p>"
                ]]},
                "mode"    : "000644",
                "owner"   : "root",
                "group"   : "root"
              },

              "/etc/httpd/conf.d/maptobackend.conf" : {
                "content" : { "Fn::Join" : ["", [
                   "ProxyPass /backend http://", { "Fn::GetAtt" : [ "PrivateElasticLoadBalancer", "DNSName" ]}, "\n",
                   "ProxyPassReverse /backend http://", { "Fn::GetAtt" : [ "PrivateElasticLoadBalancer", "DNSName" ]}, "\n"
                ]]},
                "mode"    : "000644",
                "owner"   : "root",
                "group"   : "root"
              }
            },

            "services" : {
              "sysvinit" : {
                "httpd" : {
                  "enabled"       : "true",
                  "ensureRunning" : "true",
                  "files"         : [ "/etc/httpd/conf.d/maptobackend.conf", "/var/www/html/index.html" ]
                }
              }
            }
          }
        }
      },
      "Properties" : {
        "ImageId"        : { "Fn::FindInMap" : [ "AWSRegionArch2AMI", { "Ref" : "AWS::Region" }, { "Fn::FindInMap" : [ "AWSInstanceType2Arch", { "Ref" : "FrontendInstanceType" }, "Arch" ] } ] },
        "SecurityGroups" : [ { "Ref" : "FrontendSecurityGroup" } ],
        "InstanceType"   : { "Ref" : "FrontendInstanceType" },
        "KeyName"        : { "Ref" : "KeyName" },
        "UserData"       : { "Fn::Base64" : { "Fn::Join" : ["", [
          "#!/bin/bash -v\n",
          "yum update -y aws-cfn-bootstrap\n",

          "# Install Apache and configure as a reverse Frontend\n",
          "/opt/aws/bin/cfn-init --stack ", { "Ref" : "AWS::StackName" }, " --resource FrontendServerLaunchConfig ",
          "    --access-key ",  { "Ref" : "HostKeys" },
          "    --secret-key ", { "Fn::GetAtt": ["HostKeys", "SecretAccessKey"]},
          "    --region ", { "Ref" : "AWS::Region" }, "\n",

          "# Signal completion\n",
          "/opt/aws/bin/cfn-signal -e $? -r \"Frontend setup done\" '", { "Ref" : "FrontendWaitHandle" }, "'\n"
        ]]}}
      }
    },

    "FrontendSecurityGroup" : {
      "Type" : "AWS::EC2::SecurityGroup",
      "Properties" : {
        "GroupDescription" : "Allow access from load balancer and bastion as well as outbound HTTP and HTTPS traffic",
        "VpcId" : { "Ref" : "VPC" },
        "SecurityGroupIngress" : [
          { "IpProtocol" : "tcp", "FromPort" : "80", "ToPort" : "80", "SourceSecurityGroupId" : { "Ref" : "PublicLoadBalancerSecurityGroup" } } ,
          { "IpProtocol" : "tcp", "FromPort" : "22", "ToPort" : "22", "SourceSecurityGroupId" : { "Ref" : "BastionSecurityGroup" } } ],
        "SecurityGroupEgress" : [
           { "IpProtocol" : "tcp", "FromPort" : "80",  "ToPort" : "80",  "CidrIp" : "0.0.0.0/0" } ,
           { "IpProtocol" : "tcp", "FromPort" : "443", "ToPort" : "443", "CidrIp" : "0.0.0.0/0" } ]
      }
    },

    "FrontendWaitHandle" : {
      "Type" : "AWS::CloudFormation::WaitConditionHandle"
    },

    "FrontendWaitCondition" : {
      "Type" : "AWS::CloudFormation::WaitCondition",
      "DependsOn" : "FrontendFleet",
      "Properties" : {
        "Handle"  : { "Ref" : "FrontendWaitHandle" },
        "Timeout" : "300",
        "Count"   : { "Ref" : "FrontendSize" }
      }
    },


    "PrivateElasticLoadBalancer" : {
      "Type" : "AWS::ElasticLoadBalancing::LoadBalancer",
      "Properties" : {
        "SecurityGroups" : [ { "Ref" : "PrivateLoadBalancerSecurityGroup" } ],
        "Subnets" : [ { "Ref" : "PrivateSubnet" } ],
        "Listeners" : [ { "LoadBalancerPort" : "80", "InstancePort" : "80", "Protocol" : "HTTP" } ],
        "Scheme" : "internal",
        "HealthCheck" : {
          "Target" : "HTTP:80/index.html",
          "HealthyThreshold" : "3",
          "UnhealthyThreshold" : "5",
          "Interval" : "90",
          "Timeout" : "60"
        }
      }
    },

    "PrivateLoadBalancerSecurityGroup" : {
      "Type" : "AWS::EC2::SecurityGroup",
      "Properties" : {
        "GroupDescription" : "Private ELB Security Group with HTTP access on port 80 from the Frontend Fleet only",
        "VpcId" : { "Ref" : "VPC" },
        "SecurityGroupIngress" : [ { "IpProtocol" : "tcp", "FromPort" : "80", "ToPort" : "80", "SourceSecurityGroupId" : { "Ref" : "FrontendSecurityGroup" } } ],
        "SecurityGroupEgress" : [ { "IpProtocol" : "tcp", "FromPort" : "80", "ToPort" : "80", "CidrIp" : "0.0.0.0/0" } ]
      }
    },

    "BackendFleet" : {
      "Type" : "AWS::AutoScaling::AutoScalingGroup",
      "Properties" : {
        "AvailabilityZones" : [{ "Fn::GetAtt" : [ "PrivateSubnet", "AvailabilityZone" ] }],
        "VPCZoneIdentifier" : [{ "Ref" : "PrivateSubnet" }],
        "LaunchConfigurationName" : { "Ref" : "BackendLaunchConfig"  },
        "MinSize" : "1",
        "MaxSize" : "10",
        "DesiredCapacity" : { "Ref" : "BackendSize" },
        "LoadBalancerNames" : [ { "Ref" : "PrivateElasticLoadBalancer" } ],
        "Tags" : [ { "Key" : "Network", "Value" : "Private", "PropagateAtLaunch" : "true" } ]
      }
    },

    "BackendLaunchConfig"  : {
      "Type" : "AWS::AutoScaling::LaunchConfiguration",
      "Metadata" : {
        "Comment1" : "Configure the Backend server to respond to requests",

        "AWS::CloudFormation::Init" : {
          "config" : {
            "packages" : {
              "yum" : {
                "mailx"            : [],
                "lynx"             : [],
                "mysql"            : [],
                "mysql-server"     : [],
                "mysql-libs"       : [],
                "httpd"            : [],
                "php"              : [],
                "php-mysql"        : [],
                "php-common"       : [],
                "php-cli"          : [],
                "php-gd"           : [],
                "php-xml"          : [],
                "php-mbstring"     : [],
                "mysql-devel"      : [],
                "php-pecl-memcache" : [],
                "ImageMagick"      : [],
                "memcached"        : []              }
            },

            "files" : {
              "/var/www/html/index.html" : {
                "content" : { "Fn::Join" : ["\n", [
                   "<img src=\"https://s3.amazonaws.com/cloudformation-examples/cloudformation_graphic.png\" alt=\"AWS CloudFormation Logo\"/>",
                   "<h1>Congratulations, this request was served from the backend fleet</h1>"
                ]]},
                "mode"    : "000644",
                "owner"   : "root",
                "group"   : "root"
              }
            },

            "services" : {
              "sysvinit" : {
                "httpd" : {
                  "enabled"       : "true",
                  "ensureRunning" : "true",
                  "files"         : [ "/var/www/html/index.html" ]
                }
              }
            }
          }
        }
      },
      "Properties" : {
        "ImageId"        : { "Fn::FindInMap" : [ "AWSRegionArch2AMI", { "Ref" : "AWS::Region" }, { "Fn::FindInMap" : [ "AWSInstanceType2Arch", { "Ref" : "BackendInstanceType" }, "Arch" ] } ] },
        "SecurityGroups" : [ { "Ref" : "BackendSecurityGroup" } ],
        "InstanceType"   : { "Ref" : "BackendInstanceType" },
        "KeyName"        : { "Ref" : "KeyName" },
        "UserData"       : { "Fn::Base64" : { "Fn::Join" : ["", [
          "#!/bin/bash -v\n",
          "yum update -y aws-cfn-bootstrap\n",

          "# Install Apache\n",
          "/opt/aws/bin/cfn-init --stack ", { "Ref" : "AWS::StackName" }, " --resource BackendLaunchConfig ",
          "    --access-key ",  { "Ref" : "HostKeys" },
          "    --secret-key ", { "Fn::GetAtt": ["HostKeys", "SecretAccessKey"]},
          "    --region ", { "Ref" : "AWS::Region" }, "\n",

          "# Signal completion\n",
          "/opt/aws/bin/cfn-signal -e $? -r \"Backend setup done\" '", { "Ref" : "BackendWaitHandle" }, "'\n"
        ]]}}
      }
    },

    "BackendSecurityGroup" : {
      "Type" : "AWS::EC2::SecurityGroup",
      "Properties" : {
        "GroupDescription" : "Allow access from private load balancer and bastion as well as outbound HTTP and HTTPS traffic",
        "VpcId" : { "Ref" : "VPC" },
        "SecurityGroupIngress" : [
          { "IpProtocol" : "tcp", "FromPort" : "80", "ToPort" : "80", "SourceSecurityGroupId" : { "Ref" : "PrivateLoadBalancerSecurityGroup" } } ,
          { "IpProtocol" : "tcp", "FromPort" : "22", "ToPort" : "22", "SourceSecurityGroupId" : { "Ref" : "BastionSecurityGroup" } } ],
        "SecurityGroupEgress" : [
           { "IpProtocol" : "tcp", "FromPort" : "80",  "ToPort" : "80",  "CidrIp" : "0.0.0.0/0" } ,
           { "IpProtocol" : "tcp", "FromPort" : "443", "ToPort" : "443", "CidrIp" : "0.0.0.0/0" } ]
      }
    },

    "BackendWaitHandle" : {
      "Type" : "AWS::CloudFormation::WaitConditionHandle"
    },

    "BackendWaitCondition" : {
      "Type" : "AWS::CloudFormation::WaitCondition",
      "DependsOn" : "BackendFleet",
      "Properties" : {
        "Handle"  : { "Ref" : "BackendWaitHandle" },
        "Timeout" : "300",
        "Count"   : { "Ref" : "BackendSize" }
      }
    }
  },

  "Outputs" : {
    "WebSite" : {
      "Description" : "URL of the website",
      "Value" :  { "Fn::Join" : [ "", [ "http://", { "Fn::GetAtt" : [ "PublicElasticLoadBalancer", "DNSName" ]}]]}
    },
    "Bastion" : {
      "Description" : "IP Address of the Bastion host",
      "Value" :  { "Ref" : "BastionIPAddress" }
    },
    "myLBDNSRecord" : {
      "Description" : "DNS Record for the frontend loadbalancer.",
      "Value" :  { "Ref" : "myLBDNSRecord" }
    }
  }
}