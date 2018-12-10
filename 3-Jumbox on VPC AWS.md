#Deploying a bastion host in AWS cloud, setting up a NAT server and accessing private EC2 instances. 

References: 
http://justsomestuff.co.uk/theblog/2017/01/15/using-a-bastion-host-to-access-your-aws-ec2-instances/
https://aws.amazon.com/blogs/security/securely-connect-to-linux-instances-running-in-a-private-amazon-vpc/

## Summary of Settings 
----------------------------------------------------------
VPC IP (10.10.0.0/16) 		      

Subnets 
  - Public Subnet (10.10.0/24)
    - Auto Assign IP 
  - Private Subnet:(10.10.1.0/24)
  
Internet Gateway - Attached to VPC

Route Tables
  - Public 
    - Destination 
      - 0.0.0.0/0
          - Target: Internet Gateway
  - Private 
     -Destination 
      - 0.0.0.0/0
          - Target: Nat Server Instance

Security Groups 
	- NAT SG
		- Inbound: All Traffic from Private Subnet(10.10.1.0/24)
		- Outbound: All Traffic(0.0.0.0/0) 
	- Bastion SG
		- Inbound: SSH from my IP
		- Outbout: All Traffic 
	- Internal SG (Private) 
		- Inbound: SSH from Bastion SG
		- Outbond: All Traffic  
		

## Instances
	(1) NAT Server  
			- Subnet:Public
			- IP:Auto (Default) 
		- Advanced Details(Input User Script). Configures NAT by enabling port forwarding and IPV4 masquerading to make external requeststs when the service is launched.
							#!/bin/sh
							echo 1 > /proc/sys/net/ipv4/ip_forward  #Enable IPV4 port forwarding                             
							echo 0 > /proc/sys/net/ipv4/conf/eth0/send_redirects 	#Security precaution to stop optimal path route cache entry
							/sbin/iptables -t nat -A POSTROUTING -o eth0 -s 0.0.0.0/0 -j MASQUERADE #Masks the private IP
							/sbin/iptables-save > /etc/sysconfig/iptables # saves the settings 
							mkdir -p /etc/sysctl.d/ 
							cat <<EOF > /etc/sysctl.d/nat.conf #File to save nat config whenever instance is rebooted
							net.ipv4.ip_forward = 1 
							net.ipv4.conf.eth0.send_redirects = 0
							EOF 
			- Disable Change Source/Dest -  prevent AWS from rejecting IP packets that are not directly addressed to the NAT server instance’s 				 IP address
		(2) Bastion Host
			- IP: Auto 
			- Advanced Details(Input Script) - Ensures server has latest security 0atches 
							#!/bin/sh
							yum update -y
		(3) Internal Instance 
			- IP: Disabled
			
Key Pairs
-(1)NAT & Basiton Key pairs
-(2)Private Instance Key Pair 

## Connection Guide
----------------------------------------------------------			
1. Generate key pairs with  PuttyGen 
2. Install Paegant(from putty website) - this allows keys to be authenticated from your device without uploading to instance
3. Add Security Keys to Paegant
4. Launch Putty, In Auth leave file blank and enable agent forwarding(peagant) 
5. Connect to Bastion Host(Public IP) 
6. Connect to Private Instance and Ping 8.8.8.8

## Results
------------------------------------------------------------
login as: ec2-user
Authenticating with public key "imported-openssh-key" from agent
Last login: Sat Dec  1 20:30:52 2018 from 37.173.73.5

       __|  __|_  )
       _|  (     /   Amazon Linux AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-ami/2018.03-release-notes/
[ec2-user@ip-10-10-0-205 ~]$ ssh ec2-user@10.10.1.195
The authenticity of host '10.10.1.195 (10.10.1.195)' can't be established.
ECDSA key fingerprint is SHA256:Nd5D5FkUN+RlN9Civ3oRVsvatKkFJ0Pedcb58+0H2XE.
ECDSA key fingerprint is MD5:07:cf:48:98:c2:3f:4a:25:08:a2:ac:26:3a:79:04:61.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.1.195' (ECDSA) to the list of known hosts.

       __|  __|_  )
       _|  (     /   Amazon Linux AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-ami/2018.03-release-notes/
[ec2-user@ip-10-10-1-195 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=108 time=11.4 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=108 time=11.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=108 time=11.4 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=108 time=11.5 ms
