# cloud-formation


{
    "AWSTemplateFormatVersion": "2010-09-09",
    "Description": "Nested Stack",
    "Metadata": {
         "About": {
             "Author": {"value": "DevOps Engineering Team"},
             "CreatedDate": {"Value": "October 18, 2017"},
             "Contact": {"Value": "cwdsdevopsengineering@osi.ca.gov"},
             "UpdatedDate": {"Value": "April 17, 2018"}
         }
    },
	"Parameters": {
		"DBPassword": {
			"Description": "Postgres Database admin account Password",
            "NoEcho": "true",
            "Type": "String",
            "MinLength": "1",
            "MaxLength": "20",
            "ConstraintDescription": "Must Begin with a letter and contain only alphanumeric characters."
		},
        "ProxyNetwork": {
            "Type": "String",
            "Default": "proxy2",
            "AllowedValues" : ["proxy00", "proxy1", "proxy2"],
            "Description": "Select the Proxy Peering Network Environment"
        }
    },
	"Mappings":{
		"CustomVariables": { 
			"EnvName": {"Value": "test"},
			"KeyName": {"Value":"cwds_preint_keypair"},
			"RegionAZNameA": {"Value":"us-west-1a"},
			"RegionAZNameC": {"Value":"us-west-1c"},
			"VpcCidrBlock": {"Value":"10.63.0.0/16"},
			"PublicCidr2A": {"Value":"10.63.2.0/24"},
			"PublicCidr2C": {"Value":"10.63.3.0/24"},
			"PrivateCidr1A": {"Value":"10.63.10.0/24"},
			"PrivateCidr1C": {"Value":"10.63.11.0/24"},
			"PrivateCidr2A": {"Value":"10.63.12.0/24"},
			"PrivateCidr2C": {"Value":"10.63.13.0/24"},
			"PrivateCidr3A": {"Value":"10.63.14.0/24"},				
			"ImageID": {"Value":"ami-1dd8fa7d"},
			"SSLCertificateId": {"Value": "arn:aws:acm:us-west-1:142152461930:certificate/cc64a5d5-4163-4ee2-a493-94819dc00476"},
			"InstanceType": {"Value":"t2.medium"},
			"RundeckInstanceType": {"Value":"t2.medium"},
			"ElasticSearchInstanceType": {"Value":"m4.large"},
			"DisableApiTermination": {"Value": "false"},
			"OpenVpnCidr": {"Value":"10.210.2.217/32"},
            "SafCidr": {"Value": "162.2.15.101/32"},
			"HostedZoneName": {"Value": "cwds.io"}
		},

        "Conditions": {
            "ProxyVpcPeering": {"Value": "true"},
            "EnvironmentType": {"Value": "docker"},
            "ProxyWebGateway": {"Value": "false"},
			"calsdeploy": {"Value": "true"},
			"idpdeploy": {"Value": "false"},
			"calsapideploy": {"Value": "true"},
		    "doradeploy": {"Value": "true"},
			"geoapideploy": {"Value": "true"},
            "cmapideploy": {"Value": "true"},
            "createES1node": {"Value": "false"}
        },
		"WebProxygateway": {
			"ProxygatewayExternalIp": {"Value": "54.153.23.73/32"},
			"ProxygatewaySg": {"Value": "sg-35cb1852"}
		},
		"DbVariables": {
			"PostgresRDS":{"Value":"false"},
			"PostgresReadReplica":{"Value":"false"},
			"DBInstancetype": {"Value": "db.t2.small"},
			"DBAllocatedStorage": {"Value": "100"},
			"Iops": {"Value": "1000"},
			"InstanceIdentifier":{"Value":"postgres-cocina"},
			"DBUsername":{"Value":"cocina_postgres"},
			"DBName": {"Value": "DBN1SOC"},
			"ElasticSearchVolume": {"Value": "100"},
			"CacheNodeType": {"Value": "cache.t2.small"},
			"RedisCacheClusters": {"Value": "1"},
			"RedisAutomaticFailoverEnabled": {"Value": "false"}
		}
	},

     "Resources": {
	    "VPC": {
		    "Type": "AWS::CloudFormation::Stack",
			"Metadata":{ "Comment": "Creates and updates the VPC, Subnets, Routes, IGW, NAT and EIP for the Webgateway " },
			"Properties": {
			    "TemplateURL": "https://cwds-cloudformation.s3.amazonaws.com/Stable-Environments/common-templates/template-vpc.json",
				"TimeoutInMinutes": "10",
				"Parameters": {
					"EnvName": { "Fn::FindInMap": ["CustomVariables", "EnvName", "Value"]},
					"RegionAZNameA": { "Fn::FindInMap": ["CustomVariables", "RegionAZNameA", "Value"]},
					"RegionAZNameC": { "Fn::FindInMap": ["CustomVariables", "RegionAZNameC", "Value"]},
					"VpcCidrBlock": { "Fn::FindInMap": ["CustomVariables", "VpcCidrBlock", "Value"]},
					"PublicCidr2A": { "Fn::FindInMap": ["CustomVariables", "PublicCidr2A", "Value"]},
					"PublicCidr2C": { "Fn::FindInMap": ["CustomVariables", "PublicCidr2C", "Value"]},
					"PrivateCidr1A": { "Fn::FindInMap": ["CustomVariables", "PrivateCidr1A", "Value"]},
					"PrivateCidr1C": { "Fn::FindInMap": ["CustomVariables", "PrivateCidr1C", "Value"]},
					"PrivateCidr2A": { "Fn::FindInMap": ["CustomVariables", "PrivateCidr2A", "Value"]},
					"PrivateCidr2C": { "Fn::FindInMap": ["CustomVariables", "PrivateCidr2C", "Value"]},
					"PrivateCidr3A": { "Fn::FindInMap": ["CustomVariables", "PrivateCidr3A", "Value"]}
               	}
			}
        },
        "VpcPeering":{
			"Type": "AWS::CloudFormation::Stack",
			"Metadata":{ "Comment": "Perrring connection Management-curret environment and proxy-current environment" },
			"DependsOn": "VPC",
			"Properties": {
				"TemplateURL": "https://cwds-cloudformation.s3.amazonaws.com/Stable-Environments/common-templates/template-vpc-peering.json",
				"TimeoutInMinutes": "10",
				"Parameters": {
					"EnvName": { "Fn::FindInMap": ["CustomVariables", "EnvName", "Value"]},
					"VpcID": {"Fn::GetAtt": ["VPC", "Outputs.VpcID"]},
					"VpcCidrBlock": { "Fn::FindInMap": ["CustomVariables", "VpcCidrBlock", "Value"]},
					"RouteTable1": {"Fn::GetAtt": ["VPC", "Outputs.Route1"]},
					"RouteTable2A": {"Fn::GetAtt": ["VPC", "Outputs.Route2A"]},
					"RouteTable2C": {"Fn::GetAtt": ["VPC", "Outputs.Route2C"]},
					"ManagementVpcId": {"Fn::ImportValue" : {"Fn::Sub" : "Management-VPC-ID"}},
			        "ManagementVpcCidr": {"Fn::ImportValue" : {"Fn::Sub" : "Management-VPC-CIDR"}},          		
			        "ManagementRoute1": {"Fn::ImportValue" : {"Fn::Sub" : "Management-VPC-Route1"}},
                    "ManagementRoute2": {"Fn::ImportValue" : {"Fn::Sub" : "Management-VPC-Route2"}},
					"ProxyVpcPeering":{ "Fn::FindInMap": ["Conditions", "ProxyVpcPeering", "Value"]},
                    "ProxyVpcId": {"Fn::ImportValue" : {"Fn::Sub" : "${ProxyNetwork}-VPC-ID"}},
			        "ProxyVpcCidr": {"Fn::ImportValue" : {"Fn::Sub" : "${ProxyNetwork}-CidrRange"}},         		
			        "ProxyRoute1": {"Fn::ImportValue" : {"Fn::Sub" : "${ProxyNetwork}-RouteTable1-ID"}},
			        "ProxyRoute2": {"Fn::ImportValue" : {"Fn::Sub" : "${ProxyNetwork}-RouteTable2-ID"}}
				}
			}
		},
        "SecurityGroups": {
		    "Type": "AWS::CloudFormation::Stack",
			"Metadata":{ "Comment": "Creates and Updates all the Security Groups required for ec2-instances and databases" },
			"DependsOn": "VpcPeering",
			"Properties": {
			    "TemplateURL": "https://cwds-cloudformation.s3.amazonaws.com/Stable-Environments/common-templates/template-securitygroups.json",
				"TimeoutInMinutes": "10",
				"Parameters": {
					"VpcID": {"Fn::GetAtt": ["VPC", "Outputs.VpcID"]},
					"EnvName": { "Fn::FindInMap": ["CustomVariables", "EnvName", "Value"]},
		            "OpenVpnCidr": { "Fn::FindInMap": ["CustomVariables", "OpenVpnCidr", "Value"]},
                    "SafCidr": { "Fn::FindInMap": ["CustomVariables", "SafCidr", "Value"]},
					"ProxygatewayExternalIp": { "Fn::FindInMap": ["WebProxygateway", "ProxygatewayExternalIp", "Value"]},
					"ProxygatewaySg": { "Fn::FindInMap": ["WebProxygateway", "ProxygatewaySg", "Value"]},
					"NatCidrA": {"Fn::GetAtt": ["VPC", "Outputs.NatEIP2A"]},
					"NatCidrC": {"Fn::GetAtt": ["VPC", "Outputs.NatEIP2C"]},
            		"JenkinsMgmtSg": {"Fn::ImportValue" : {"Fn::Sub" : "Jenkins-master-sg"}}
               	}
			}
        },
		"CoreServers":{
		    "Type": "AWS::CloudFormation::Stack",
			"Metadata":{ "Comment": "Spinup/Updates the core ec2 - servers (Jenkins, rundeck)" },
			"DependsOn": "SecurityGroups",
			"Properties": {
				"TemplateURL": "https://cwds-cloudformation.s3.amazonaws.com/Stable-Environments/common-templates/template-coreservers.json",
				"TimeoutInMinutes": "10",
				"Parameters": {
					"EnvName": { "Fn::FindInMap": ["CustomVariables", "EnvName", "Value"]},
					"PrivateSubnet3A": {"Fn::GetAtt": ["VPC", "Outputs.PrivateSubnet3A"]},
					"KeyName": { "Fn::FindInMap": ["CustomVariables", "KeyName", "Value"]},
					"ImageID": { "Fn::FindInMap": ["CustomVariables", "ImageID", "Value"]},
					"InstanceType": { "Fn::FindInMap": ["CustomVariables", "InstanceType", "Value"]},
					"RundeckInstanceType": { "Fn::FindInMap": ["CustomVariables", "RundeckInstanceType", "Value"]},
					"DisableApiTermination": { "Fn::FindInMap": ["CustomVariables", "DisableApiTermination", "Value"]},
 		            "OpenVpnCidr": { "Fn::FindInMap": ["CustomVariables", "OpenVpnCidr", "Value"]},			
        			"ManagedServerRules": {"Fn::GetAtt": ["SecurityGroups", "Outputs.ManagedServerRules"]},
					"JenkinsSg": {"Fn::GetAtt": ["SecurityGroups", "Outputs.JenkinsSg"]},
					"Rundecksg": {"Fn::GetAtt": ["SecurityGroups", "Outputs.Rundecksg"]},
					"proxydb2sg": {"Fn::GetAtt": ["SecurityGroups", "Outputs.proxydb2sg"]}
				
		    	}
			}
		},
        "ApplicationServers":{
		    "Type": "AWS::CloudFormation::Stack",
			"Metadata":{ "Comment": "Application ec2 - servers" },
			"DependsOn": "SecurityGroups",
			"Properties": {
				"TemplateURL": "https://cwds-cloudformation.s3.amazonaws.com/Stable-Environments/common-templates/template-application-ec2.json",
				"TimeoutInMinutes": "10",
				"Parameters": {
					"EnvName": { "Fn::FindInMap": ["CustomVariables", "EnvName", "Value"]},
					"PublicSubnet2A": {"Fn::GetAtt": ["VPC", "Outputs.PublicSubnet2A"]},
					"PublicSubnet2C": {"Fn::GetAtt": ["VPC", "Outputs.PublicSubnet2C"]},
					"PrivateSubnet2A": {"Fn::GetAtt": ["VPC", "Outputs.PrivateSubnet2A"]},
        			"PrivateSubnet2C":{"Fn::GetAtt": ["VPC", "Outputs.PrivateSubnet2C"]},
					"KeyName": { "Fn::FindInMap": ["CustomVariables", "KeyName", "Value"]},
					"ImageID": { "Fn::FindInMap": ["CustomVariables", "ImageID", "Value"]},
					"InstanceType": { "Fn::FindInMap": ["CustomVariables", "InstanceType", "Value"]},
					"DisableApiTermination": { "Fn::FindInMap": ["CustomVariables", "DisableApiTermination", "Value"]},
					"SSLCertificateId": { "Fn::FindInMap": ["CustomVariables", "SSLCertificateId", "Value"]},
                    "ProxyWebGateway":{ "Fn::FindInMap": ["Conditions", "ProxyWebGateway", "Value"]},
					"calsdeploy":{ "Fn::FindInMap": ["Conditions", "calsdeploy", "Value"]},
					"idpdeploy": { "Fn::FindInMap": ["Conditions", "idpdeploy", "Value"]},
					"calsapideploy":{ "Fn::FindInMap": ["Conditions", "calsapideploy", "Value"]},		
					"doradeploy":{ "Fn::FindInMap": ["Conditions", "doradeploy", "Value"]},	
					"cmapideploy":{ "Fn::FindInMap": ["Conditions", "cmapideploy", "Value"]},
        			"geoapideploy":{ "Fn::FindInMap": ["Conditions", "geoapideploy", "Value"]},
					"ManagedServerRules":{"Fn::GetAtt": ["SecurityGroups", "Outputs.ManagedServerRules"]},
        			"WebProxyGatewaySg": {"Fn::GetAtt": ["SecurityGroups", "Outputs.WebProxyGatewaySg"]},	
					"WebProxyGatewaySg2": {"Fn::GetAtt": ["SecurityGroups", "Outputs.WebProxyGatewaySg2"]},
					"WebGatewayElbSg": 	{"Fn::GetAtt": ["SecurityGroups", "Outputs.WebGatewayElbSg"]},
					"WebAppsg":{"Fn::GetAtt": ["SecurityGroups", "Outputs.WebAppsg"]},
					"Appsg": {"Fn::GetAtt": ["SecurityGroups", "Outputs.Appsg"]},
			  		"Perrysg":{"Fn::GetAtt": ["SecurityGroups", "Outputs.Perrysg"]},
					"IDPsg":{"Fn::GetAtt": ["SecurityGroups", "Outputs.IDPsg"]},
					"WebPerryExternalsg":{"Fn::GetAtt": ["SecurityGroups", "Outputs.WebPerryExternalsg"]},
					"Apisg":{"Fn::GetAtt": ["SecurityGroups", "Outputs.Apisg"]},   
					"WebApisg": {"Fn::GetAtt": ["SecurityGroups", "Outputs.WebApisg"]},
					"WebApisg2":{"Fn::GetAtt": ["SecurityGroups", "Outputs.WebApisg2"]},
					"proxydb2sg": {"Fn::GetAtt": ["SecurityGroups", "Outputs.proxydb2sg"]}
				
		    	}
			}
		},
		"Databases": {
			"Type": "AWS::CloudFormation::Stack",
			"Metadata":{ "Comment": "creates/updates the databases" },
			"DependsOn": "SecurityGroups",
			"Properties": {
				"TemplateURL": "https://cwds-cloudformation.s3.amazonaws.com/Stable-Environments/common-templates/template-databases.json",
				"Parameters": {
					"EnvName": { "Fn::FindInMap": ["CustomVariables", "EnvName", "Value"]},
					"PrivateSubnet1A": {"Fn::GetAtt": ["VPC", "Outputs.PrivateSubnet1A"]},
					"PrivateSubnet1C":{"Fn::GetAtt": ["VPC", "Outputs.PrivateSubnet1C"]},
					"KeyName": { "Fn::FindInMap": ["CustomVariables", "KeyName", "Value"]},
					"ImageID": { "Fn::FindInMap": ["CustomVariables", "ImageID", "Value"]},
					"InstanceType": { "Fn::FindInMap": ["CustomVariables", "InstanceType", "Value"]},
					"ElasticSearchInstanceType": { "Fn::FindInMap": ["CustomVariables", "ElasticSearchInstanceType", "Value"]},
                    "createES1node": { "Fn::FindInMap": ["Conditions", "createES1node", "Value"]},
					"EnvironmentType":{ "Fn::FindInMap": ["Conditions", "EnvironmentType", "Value"]},
					"calsdeploy":{ "Fn::FindInMap": ["Conditions", "calsdeploy", "Value"]},	
					"DisableApiTermination": { "Fn::FindInMap": ["CustomVariables", "DisableApiTermination", "Value"]},
					"ManagedServerRules":{"Fn::GetAtt": ["SecurityGroups", "Outputs.ManagedServerRules"]},
					"postgressg": {"Fn::GetAtt": ["SecurityGroups", "Outputs.postgressg"]},
					"Calspgsg": {"Fn::GetAtt": ["SecurityGroups", "Outputs.Calspgsg"]},
        			"db2sg":{"Fn::GetAtt": ["SecurityGroups", "Outputs.db2sg"]},
					"elasticsearchsg": {"Fn::GetAtt": ["SecurityGroups", "Outputs.elasticsearchsg"]},
					"Redissg":{"Fn::GetAtt": ["SecurityGroups", "Outputs.Redissg"]},
					"ElasticSearchVolume": { "Fn::FindInMap": ["DbVariables", "ElasticSearchVolume", "Value"]},
					"PostgresReadReplica":{ "Fn::FindInMap": ["DbVariables", "PostgresReadReplica", "Value"]},
					"DBInstancetype": { "Fn::FindInMap": ["DbVariables", "DBInstancetype", "Value"]},
					"DBAllocatedStorage":{ "Fn::FindInMap": ["DbVariables", "DBAllocatedStorage", "Value"]},
					"Iops":{ "Fn::FindInMap": ["DbVariables", "Iops", "Value"]},
					"InstanceIdentifier":{ "Fn::FindInMap": ["DbVariables", "InstanceIdentifier", "Value"]},
					"DBUsername":{ "Fn::FindInMap": ["DbVariables", "DBUsername", "Value"]},
					"DBPassword": {"Ref": "DBPassword"},
					"DBName": { "Fn::FindInMap": ["DbVariables", "DBName", "Value"]},
					"CacheNodeType": { "Fn::FindInMap": ["DbVariables", "CacheNodeType", "Value"]},
					"RedisCacheClusters": {"Fn::FindInMap": ["DbVariables", "RedisCacheClusters", "Value"]},
					"RedisAutomaticFailoverEnabled": {"Fn::FindInMap": ["DbVariables", "RedisAutomaticFailoverEnabled", "Value"]}
				}
			}	
		},
		"Alarms": {
			"Type": "AWS::CloudFormation::Stack",
			"Metadata":{ "Comment": "Alarms for All the Servers in the Environment" },
			"DependsOn": "Databases",
			"Properties": {
				"TemplateURL": "https://cwds-cloudformation.s3.amazonaws.com/Stable-Environments/common-templates/template-alarms.json",
				"TimeoutInMinutes": "10",
				"Parameters": {
					"EnvName": { "Fn::FindInMap": ["CustomVariables", "EnvName", "Value"]},
					"InitialWarning": {"Fn::ImportValue" : {"Fn::Sub" : "SNSAlertWarning"}},
					"CriticalWarning": {"Fn::ImportValue" : {"Fn::Sub" : "SNSAlertCritical"}},
					"EnvironmentType":{ "Fn::FindInMap": ["Conditions", "EnvironmentType", "Value"]},
					"ProxyWebGateway":{ "Fn::FindInMap": ["Conditions", "ProxyWebGateway", "Value"]},
					"InternalWebGatewayAInstanceId": {"Fn::GetAtt": ["ApplicationServers", "Outputs.InternalWebGatewayAInstanceId"]},
					"InternalWebGatewayCInstanceId": {"Fn::GetAtt": ["ApplicationServers", "Outputs.InternalWebGatewayCInstanceId"]},
					"AppServerAInstanceId": {"Fn::GetAtt": ["ApplicationServers", "Outputs.AppServerAInstanceId"]},
					"AppServerCInstanceId": {"Fn::GetAtt": ["ApplicationServers", "Outputs.AppServerCInstanceId"]},
					"PerryServerAInstanceId": {"Fn::GetAtt": ["ApplicationServers", "Outputs.PerryServerAInstanceId"]},
					"PerryServerCInstanceId": {"Fn::GetAtt": ["ApplicationServers", "Outputs.PerryServerCInstanceId"]},
					"ApiServerAInstanceId": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ApiServerAInstanceId"]},
					"ApiServerCInstanceId": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ApiServerCInstanceId"]},
					"RundeckServerAInstanceId": {"Fn::GetAtt": ["CoreServers", "Outputs.RundeckServerAInstanceId"]},
					"JenkinsInstanceId": {"Fn::GetAtt": ["CoreServers", "Outputs.JenkinsInstanceId"]},
					"DB2AInstanceId": {"Fn::GetAtt": ["Databases", "Outputs.DB2AInstanceId"]},
					"PostgreSQLAInstanceId": {"Fn::GetAtt": ["Databases", "Outputs.PostgreSQLAInstanceId"]},
					"ElasticSearchInstanceId": {"Fn::GetAtt": ["Databases", "Outputs.ElasticSearchInstanceId"]},
                    "createES1node": { "Fn::FindInMap": ["Conditions", "createES1node", "Value"]}
					
				}
			}
		},
		"Route53": {
			"Type": "AWS::CloudFormation::Stack",
			"Metadata":{ "Comment": "Creates/Updates recordsets in the environment" },
			"DependsOn": "Databases",
			"Properties": {
				"TemplateURL": "https://cwds-cloudformation.s3.amazonaws.com/Stable-Environments/common-templates/template-route53.json",
				"TimeoutInMinutes": "10",
				"Parameters": {
					"EnvName": { "Fn::FindInMap": ["CustomVariables", "EnvName", "Value"]},
					"EnvironmentType":{ "Fn::FindInMap": ["Conditions", "EnvironmentType", "Value"]},
					"ProxyWebGateway":{ "Fn::FindInMap": ["Conditions", "ProxyWebGateway", "Value"]},
					"calsdeploy":{ "Fn::FindInMap": ["Conditions", "calsdeploy", "Value"]},
					"idpdeploy":{ "Fn::FindInMap": ["Conditions", "idpdeploy", "Value"]},
					"calsapideploy":{ "Fn::FindInMap": ["Conditions", "calsapideploy", "Value"]},
					"doradeploy":{ "Fn::FindInMap": ["Conditions", "doradeploy", "Value"]},
					"geoapideploy":{ "Fn::FindInMap": ["Conditions", "geoapideploy", "Value"]},
					"HostedZoneName": { "Fn::FindInMap": ["CustomVariables", "HostedZoneName", "Value"]},
					"InternalWebGatewayA": {"Fn::GetAtt": ["ApplicationServers", "Outputs.InternalWebGatewayA"]},
					"InternalWebGatewayC": {"Fn::GetAtt": ["ApplicationServers", "Outputs.InternalWebGatewayC"]},
					"ELBWebgateway": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ELBWebgateway"]},
					"AppServerA": {"Fn::GetAtt": ["ApplicationServers", "Outputs.AppServerA"]},
					"AppServerC": {"Fn::GetAtt": ["ApplicationServers", "Outputs.AppServerC"]},
					"ElbIntake": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbIntake"]},
					"ElbCals": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbCals"]},
					"ElbDashboard": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbDashboard"]},
					"PerryServerA": {"Fn::GetAtt": ["ApplicationServers", "Outputs.PerryServerA"]},
					"PerryServerC": {"Fn::GetAtt": ["ApplicationServers", "Outputs.PerryServerC"]},
					"ElbExternalPerry": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbExternalPerry"]},
					"ElbIDP": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbIDP"]},
                    "nuxeoELB": {"Fn::GetAtt": ["ApplicationServers", "Outputs.nuxeoELB"]},
                    "dmsELB": {"Fn::GetAtt": ["ApplicationServers", "Outputs.dmsELB"]},
                    "formsapiELB": {"Fn::GetAtt": ["ApplicationServers", "Outputs.formsapiELB"]},
					"ApiServerA": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ApiServerA"]},
					"ApiServerC": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ApiServerC"]},
					"ElbIntakeApi": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbIntakeApi"]},
					"ElbCalsApi": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbCalsApi"]},
					"ElbCalsMockApi": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbCalsMockApi"]},
					"ElbFerbApi": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbFerbApi"]},
					"ElbDoraApi": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbDoraApi"]},
					"ElbGeoApi": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbGeoApi"]},
                    "ElbCase": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbCase"]},
                    "ElbCmapi": {"Fn::GetAtt": ["ApplicationServers", "Outputs.ElbCmapi"]},
					"Jenkins": {"Fn::GetAtt": ["CoreServers", "Outputs.Jenkins"]},
					"RundeckServerA": {"Fn::GetAtt": ["CoreServers", "Outputs.RundeckServerA"]},
					"PostgresEndPoint": {"Fn::GetAtt": ["Databases", "Outputs.PostgresEndPoint"]},
					"DB2A": {"Fn::GetAtt": ["Databases", "Outputs.DB2A"]},
					"PostgreSQLA": {"Fn::GetAtt": ["Databases", "Outputs.PostgreSQLA"]},
					"ElasticSearch": {"Fn::GetAtt": ["Databases", "Outputs.ElasticSearch"]},
                    "createES1node": { "Fn::FindInMap": ["Conditions", "createES1node", "Value"]},
					"IntakeRedisPrimaryEndPoint": {"Fn::GetAtt": ["Databases", "Outputs.IntakeRedisPrimaryEndPoint"]},
					"CalsRedisPrimaryEndPoint": {"Fn::GetAtt": ["Databases", "Outputs.CalsRedisPrimaryEndPoint"]}

				}
			}
		}

	}
}
