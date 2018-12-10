# create an Apache web server using an AWS instance

Create a simple Apache Webpage with a public IP.

## References:

https://docs.aws.amazon.com/en_en/AmazonRDS/latest/UserGuide/CHAP_Tutorials.WebServerDB.CreateWebServer.html

https://www.putty.org/

## 1st Task : Create your VPC
	Services -> VPC(in the left) -> Launch VPC Wizard
	
![VPC](Lab1-webpage/AWS_VPC.PNG)

![Private subnet](Lab1-webpage/Private_Subnet.PNG)

![Public subnet](Lab1-webpage/Public_Subnet.PNG)

![VPC diagram](Lab1-webpage/VPC_diagram.jpg)

## 2nd : Create an Internet Gateway and link it to the right VPC


## 3rd : Create a Route Table 
	- Go to Routes -> Edit Routes -> Add route -> 0.0.0.0/0 and choose the right "igw-....."
	- Go to Subnet Asoociation -> Edit -> choose the right subnet
![Route table](Lab1-webpage/Route_table.PNG)

## 4th : Create a Security Group
	- Go to Security Group (in the left) -> Create security group -> Description "Enable HTTP access" 
	- For clarity put a name in "Name" -> Inbound Rules -> 
	- Add HTTP (80) Anywhere 0.0.0.0/0
	- Add SSH (22) Anywhere

![Security group](Lab1-webpage/Inbound_security_group.PNG)

## 5th : Launch the EC2 instance 
	- Service -> EC2 -> 
	-Create Instance -> Select "Amazon Linux 2 AMI (HVM), SSD Volume Type" -> t2.micro 
	- Configure Instance Details -> Select the network with the right VPC -> select the subnet
	- Select Enable IP Assignation
	- add tags : Name, WebPageExample
	- Configure Security Group -> choose the right one -> Launch

![Instance details](Lab1-webpage/Instance_details.PNG)

## 6th : Puttygen and Putty (only for windows)
	- Open PuttyGen -> Load -> put the path of the key -> Save Private Key
	- Open Putty -> Session -> Fill the box of IP adress 
	- Go to SSH -> Auth -> Give the right path of the right private PPK key -> Open 
	- login as : ec2-user

## 7th : Put the rith command query in order :
	 sudo yum update -y    // for update to the last version and -y stands for "without asking permission"
	sudo yum install -y httpd php
	sudo service httpd start
	sudo chkconfig httpd on*
	sudo groupadd www
	sudo usermod -a -G www ec2-user     // give access to www for ec2-user 
	exit // for the refresh

Then log back and query "groups" to see if the groups had been refreshed
	sudo chown -R root:www /var/www      //  Change the group ownership of the /var/www directory and its contents to the www group:

![Apache webpage](Lab1-webpage/Apache_webpage.PNG)
