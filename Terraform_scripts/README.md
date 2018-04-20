Terraform scripts for deploying project in AWS

Pre-requisite:
1. AWS account with command line access (Access ID & Key) and appropriate permissions.
2. AWS-CLI installed & configured to use the Access ID & Key.
3. Terraform installed from https://www.terraform.io/downloads.html

To deploy the terraform code, go into the directory and do the following:
1. terraform get 
2. terraform init (Required to be done only the 1st time you try to deploy using the code).
2. terraform validate
3. Execute script deploy.sh

To destroy the infrastructure, use the command "terraform destroy"


The Splunk server, MHN server, and Suricata server are in a VPC with IP 192.168.1.0/24 in us-east-1 (N.Virginia). The IP and region are mentioned in the variables.tf file in the main folder under the variable name vpc_private_cidr and private_region respectively. The subnet associated is 192.168.1.0/26 under the variable name vpc_private_subnet in the same folder.

The honeypot servers are in a separate VPC’s in the following regions:
VPC with IP 192.168.2.0/24 in us-west-1 (California)
VPC with IP 192.168.3.0/24 in ap-northeast-1 (Tokyo)
VPC with IP 192.168.4.0/24 in ap-south-1 (Mumbai)
VPC with IP 192.168.5.0/24 in ap-southeast-1 (Singapore)
VPC with IP 192.168.6.0/24 in sa-east-1 (Sao Paulo)
VPC with IP 192.168.7.0/24 in eu-west-2 (London)

The honeypot VPC in different regions peer to the honeypot in us-east-1 via VPC Peering Connection.

The subnet associated with the honeynet VPC’s in their respective regions are as follows:
The subnet for VPC with IP 192.168.2.0/24 in us-west-1 is 192.168.2.0/26
The subnet for VPC with IP 192.168.3.0/24 in ap-northeast-1 is 192.168.3.0/26
The subnet for VPC with IP 192.168.4.0/24 in ap-south-1 is 192.168.4.0/26
The subnet for VPC with IP 192.168.5.0/24 in ap-southeast-1 is 192.168.5.0/26
The subnet for VPC with IP 192.168.6.0/24 in sa-east-1 is 192.168.6.0/26
The subnet for VPC with IP 192.168.7.0/24 in eu-west-2 is 192.168.7.0/26

The honeynet VPC’s, region and subnet are mentioned in the private_ips_honeypots_cidr, honeypot_regions and private_ips_honeypots_subnet respectively in the variables.tf file in the main folder.

Note: All the VPC’s have no ACLs to them. ACLs are only used in the security groups that are assigned to individual instances. They are mentioned in the security_groups.tf file under the individual folders.

The security ACLs are as follows:
1. The Splunk server allows incoming connection from MHN, Suricata and the company (currently consisting of Public IP of project members) at specific ports. All outgoing connections are allowed.
2. The MHN server allows incoming connection from the Honeypots in various regions and the company at specific ports. All outgoing connections are allowed.
3. The Suricata server allows incoming connection from the company. All outgoing connections are allowed.
4. The Splunk server allows incoming connections from Suricata, MHN and the company at specific ports. All outgoing connections are allowed.
5. The honeypots servers are open to the world.

The public key used to connect to all these honeypots is mentioned in the public_key variable in variables.tf in the main folder.

The servers to be used to create the network infrastructure are used from the marketplace and is mentioned in the server_os.tf file in the individual folders.

Elastic IP’s are generated and attached to the MHN, Splunk and Suricata servers.
The details of the MHN, Splunk and Suricata servers, as present in the ec2.tf file, in the C&C folder are as follows:
Suricata
	Instance Type	      : t2-medium
	Volume size		      : 30GB
	OS			            : Ubuntu 14.04
  AMI			            : ubuntu-trusty-14.04*
  Suricata Setup file	: main_folder/C&C/scripts/suricata_setup.sh
  Current Private IP	: private_subnet+6 (Eg: 192.168.1.6/32)	
  Security Group	    : suricata_sg

MHN
	Instance Type	    : t2-large
	Volume size		    : 20GB
	OS			          : Ubuntu 14.04
	AMI			          : ubuntu-trusty-14.04*
	MHN Setup file	  : main_folder/C&C/scripts/mhn_setup.sh
	Current Private IP: private_subnet+4  (Eg: 192.168.1.4/32)
	Security Group	  : mhn_sg

Splunk
	Instance Type	    : c3-large
	Volume size		    : 50GB
	AMI			          : splunk_marketplace_AMI_*
	Splunk Setup file	: None
	Current Private IP: private_subnet+5  (Eg: 192.168.1.5/32)
	Security Group	  : splunk_sg

The details of the individual honeypot sensors in the honeypot servers, as present in the ec2.tf file, in the honeypots folder are as follows:
Dionaea,Cowrie,Suricata,p0f
	Instance Type	      : t2-micro
	Volume size		      : 30GB
	OS			            : Ubuntu 14.04
	AMI			            : ubuntu-trusty-14.04*
	Honeypot Setup file	: main_folder/honeypots/scripts/honeypot_setup.sh
				                main_folder/honeypots/scripts/sensor.sh
	Current Private IP	: honeypot_subnet+4 (Eg: 192.168.2.4/32)
	Security Group	    : honeypots_allow_sg
Sensors Installed	    : Dionaea, Cowrie, Suricata, p0f

Amun,Cowrie,Suricata,p0f
	Instance Type	      : t2-micro
	Volume size		      : 30GB
	OS			            : Ubuntu 14.04
	AMI			            : ubuntu-trusty-14.04*
	Honeypot Setup file	: main_folder/honeypots/scripts/honeypot_setup.sh
				                main_folder/honeypots/scripts/sensor.sh
	Current Private IP	: honeypot_subnet+5  (Eg: 192.168.2.5/32)
	Security Group	    : honeypots_allow_sg
	Sensors Installed	  : Amun, Cowrie, Suricata, p0f

ElasticHoney,Shockpot,Suricata,p0f
	Instance Type	      : t2-micro
	Volume size		      : 30GB
	OS			            : Ubuntu 14.04
	AMI			            : ubuntu-trusty-14.04*
	Honeypot Setup file	: main_folder/honeypots/scripts/honeypot_setup.sh
				                main_folder/honeypots/scripts/sensor.sh
	Current Private IP	: honeypot_subnet+6  (Eg: 192.168.2.6/32)
	Security Group	    : honeypots_allow_sg
	Sensors Installed	  : ElasticHoney, Shockpot, Suricata, p0f

Conpot,Suricata,p0f
	Instance Type	      : t2-micro
	Volume size		      : 30GB
	OS			            : Ubuntu 14.04
	AMI			            : ubuntu-trusty-14.04*
	Honeypot Setup file	: main_folder/honeypots/scripts/honeypot_setup.sh
				                main_folder/honeypots/scripts/sensor.sh
	Current Private IP	: honeypot_subnet+7 (Eg: 192.168.2.7/32)
	Security Group	    : honeypots_allow_sg
	Sensors Installed	  : Conpot, Suricata, p0f

ElasticHoney,Glastopf,Suricata,p0f
	Instance Type	      : t2-micro
	Volume size		      : 30GB
	OS			            : Ubuntu 14.04
	AMI			            : ubuntu-trusty-14.04*
	Honeypot Setup file	: main_folder/honeypots/scripts/honeypot_setup.sh
				                main_folder/honeypots/scripts/sensor.sh
	Current Private IP	: honeypot_subnet+8 (Eg: 192.168.2.8/32)
	Security Group	    : honeypots_allow_sg
  Sensors Installed	  : ElasticHoney, Glastopf, Suricata,p0f
