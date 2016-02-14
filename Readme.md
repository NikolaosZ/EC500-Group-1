# Setting up AWS to host a website

(http://docs.aws.amazon.com/gettingstarted/latest/wah-linux/web-app-hosting-intro.html)

(http://ec2-52-90-3-232.compute-1.amazonaws.com)
###Sign Up for AWS

When you sign up for Amazon Web Services (AWS), your AWS account is automatically signed up for all services in AWS and you can start using them immediately. You are charged only for the services that you use.

If you created your AWS account less than 12 months ago, you can get started with AWS for free. For more information, see AWS Free Tier.

If you have an AWS account already, skip to the next step. If you don't have an AWS account, use the following procedure to create one.

To create an AWS account

1.    Open http://aws.amazon.com/, and then choose Create an AWS Account.

2.    Follow the online instructions.

    Part of the sign-up procedure involves receiving a phone call and entering a PIN using the phone keypad.

###Create a Key Pair

AWS uses public-key cryptography to secure the login information for your instance. A Linux instance has no password; you use a key pair to log in to your instance securely. You specify the name of the key pair when you launch your instance, then provide the private key when you log in using SSH.

If you haven't created a key pair already, you can create one using the Amazon EC2 console.

To create a key pair

1.    Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.

2.    From the navigation bar, in the region selector, click US West (Oregon).

3.    In the navigation pane, click Key Pairs.

4.    Click Create Key Pair.

5.    Enter a name for the new key pair in the Key pair name field of the Create Key Pair dialog box, and then click Create. Choose a name that is easy for you to remember.

6.    The private key file is automatically downloaded by your browser. The base file name is the name you specified as the name of your key pair, and the file name extension is .pem. Save the private key file in a safe place.

    **Important**

    This is the only chance for you to save the private key file. You'll need to provide the name of your key pair when you launch an instance and the corresponding private key each time you connect to the instance.
	
###Configure a Virtual Private Cloud (VPC)

Amazon VPC enables you to launch AWS resources into a virtual network that you've defined, called a virtual private cloud (VPC). This tutorial requires the use of a VPC. Therefore, we'll check whether you already have a default VPC, and create a VPC otherwise.

**To test whether you have a default VPC**

1.    Open the Amazon VPC console at https://console.aws.amazon.com/vpc/.

2.    In the navigation bar, verify that US West (Oregon) is the selected region.

3.    In the navigation pane, click Your VPCs.

4.    One of the following is true:

        -The list is empty, so you do not have a default VPC.

        -The list has a default VPC (a VPC with a CIDR block of 172.31.0.0/16).

        -The list has one or more non-default VPCs (a VPC with a CIDR block that is not 172.31.0.0/16).

If you have a default VPC, you can use it for this tutorial, and you can skip the next procedure. Otherwise, use the following procedure to create a VPC with two public subnets for use with this tutorial.

**To create a VPC**

    1.On the VPC dashboard, click Start VPC Wizard.

    2.On the Step 1: Select a VPC Configuration page, ensure that VPC with a Single Public Subnet is selected, and click Select.

    3.On the Step 2: VPC with a Single Public Subnet page, do the following:

        a.In VPC name, enter a friendly name for your VPC.

        b.In Availability Zone, select the first Availability Zone from the list.

        c.In Subnet name, update the name from Public subnet to Public subnet 1.

        d.Leave the other default configuration settings, and click Create VPC.

        e.On the confirmation page, click OK.

    4.In the navigation pane, click Route Tables. Find the route table where the Main column is Yes. This is the main route table. Click the Name column for the main route table, enter Main, and press Enter. Click the Name column for the other route table, enter Custom, and press Enter.

    5.Add a second public subnet as follows, so that you'll have two subnets for your application servers. (Note that a default VPC already has a public subnet for each Availability Zone.)

        a.In the navigation pane, click Subnets.

        b.Click Create Subnet

        c.In Name tag, enter the name Public subnet 2.

        d.In VPC, select your VPC.

        e.In Availability Zone, select the second Availability Zone from the list.

        f.In CIDR block, enter 10.0.1.0/24.

        g.Click Yes, Create.

        h.Select the subnet named Public subnet 2, and then select the Route Table tab. Click Edit, select the route table named Custom from Change to, and then click Save. Note that this step is necessary to make this subnet a public subnet with a route to the Internet.

Next, we need to add private subnets for your database servers to your default VPC or the VPC that you just created.

**To add two private subnets to your VPC**

    1.In the navigation pane, click Subnets.

    2.Click Create Subnet

    3.In Name tag, enter the name Private subnet 1.

    4.In VPC, select your VPC.

    5.In Availability Zone, select the first Availability Zone from the list.

    6.In CIDR block, enter 10.0.2.0/24.

    7.Click Yes, Create.

    8.Click Create Subnet

    9.In Name tag, enter the name Private subnet 2.

    10.In VPC, select your VPC.

    11.In Availability Zone, select the second Availability Zone from the list.

    12.In CIDR block, enter 10.0.3.0/24.

    13.Click Yes, Create.
	
###Create a Security Group for Your Amazon EC2 Instance

A security group acts as a firewall that controls the traffic allowed to reach one or more EC2 instances. When you launch an instance, you can assign it one or more security groups. You add rules to each security group that control the traffic allowed to reach the instances to which the security group is assigned. Note that you can modify the rules for a security group at any time; the new rules take effect immediately.

For this tutorial, we'll create a security group and add the following rules:

    -Allow inbound HTTP access from anywhere

    -Allow inbound SSH traffic from your computer's public IP address so that you can connect to your instance
	
**To create and configure your security group**

    1.Decide who requires access to your instance; for example, a single computer or all the computers on a network that you trust. For this tutorial, you can use the public IP address of your computer, which you can get using a service. For example, AWS provides the following service: http://checkip.amazonaws.com. To locate another service that provides your IP address, use the search phrase "what is my IP address".

    If you are connecting through an ISP or from behind your firewall without a static IP address, you need to find out the range of IP addresses used by client computers. If you don't know this address range, you can use 0.0.0.0/0 for this tutorial. However, this is unsafe for production environments because it allows everyone to access your instance using SSH.

    2.Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.

    **Important**

    Be sure that you are using the Amazon EC2 console. If you're using the Amazon VPC console instead of the Amazon EC2 console, these directions will not match what you see.

    3.In the navigation bar, verify that US West (Oregon) is the selected region.

    4.In the navigation pane, click Security Groups, and then click Create Security Group.

    5.Enter WebServerSG as the name of the security group, and provide a description.

    6.Select your VPC from the list.

    7.On the Inbound tab, add the rules as follows:

        a.Click Add Rule, and then select SSH from the Type list. Under Source, select Custom IP and enter the public IP address range that you decided on in step 1 in the text box.

        b.Click Add Rule, and then select HTTP from the Type list.

    8.Click Create.
	
###Create an IAM Role

All requests to AWS must be cryptographically signed using credentials issued by AWS. Therefore, you need a strategy for managing credentials for software that runs on an EC2 instance. You must distribute, store, and rotate these credentials in a way that keeps them secure but also accessible to the software.

We designed IAM roles so that you can effectively manage AWS credentials for software running on your instances. You create an IAM role and configure it with the permissions that the software requires. For more information about the benefits of this approach, see IAM Roles for Amazon EC2 in the Amazon EC2 User Guide for Linux Instances and Roles (Delegation and Federation) in IAM User Guide.

The following procedure creates an IAM role that grants the web app full access to AWS. In production, you can restrict the services and resources that a web app can access.

**To create an IAM role with full access to AWS**

    1.Open the IAM console at https://console.aws.amazon.com/iam/.

    2.In the navigation pane, click Roles, and then click Create New Role.

    3.On the Set Role Name page, enter a name for the role, and then click Next Step. Remember this name, as you'll need it when you launch your instance.

    4.On the Select Role page, under AWS Service Roles, select Amazon EC2.

    5.On the Attach Policy page, select the PowerUserAccess policy, and then click Next Step.

    6.Review the role information and then click Create Role.
	
###Launch Your EC2 Instance

**To launch an EC2 instance**

    1.Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.

    2.In the navigation bar, verify that US West (Oregon) is the selected region.

    3.In the navigation pane, click Instances, and then click Launch Instance.

    4.On the Choose an Amazon Machine Image page, click Free tier only and then select an Amazon Linux AMI with the HVM virtualization type.

    5.On the Choose an Instance Type page, the t2.micro instance is selected by default. To stay within the free tier, keep this instance type. Click Next: Configure Instance Details.

    6.On the Configure Instance Details page, do the following:

        a.T2 instances must be launched into a subnet. Select your VPC from Network and select one of your public subnets from Subnet.

        b.Ensure that for Auto-assign Public IP, Enable is selected in the list. Otherwise, your instance will not get a public IP address or a public DNS name.

        c.Select your IAM role from IAM role. Note that you must select an IAM role when you launch an instance; you can't add a role to a running instance.

        d.Click Review and Launch. If you are prompted to specify the type of root volume, make your selection and then click Next.

    7.On the Review Instance Launch page, click Edit security groups. On the Configure Security Group page, click Select an existing security group, select the WebServerSG security group that you created, and then click Review and Launch.

    8.On the Review Instance Launch page, click Launch.

    9.In the Select an existing key pair or create a new key pair dialog box, select Choose an existing key pair, then select the key pair you created in Setting Up to Host a Web App on AWS. Click the acknowledgment check box, and then click Launch Instances.

    10.In the navigation pane, click Instances to see the status of your instance. Initially, the status of your instance is pending. After the status changes to running, your instance is ready for use.
	
###Connect to Your Linux Instance

After you launch your instance, you can connect to it and use it the way that you'd use a computer sitting in front of you.

Before you connect to your instance, get the public DNS name of the instance using the Amazon EC2 console. Select the instance and locate Public DNS on the Description tab.

**Tip**

If your instance doesn't have a public DNS name, open the VPC console, select the VPC, and check the Summary tab. If either DNS resolution or DNS hostnames is no, click Edit and change the value to yes.

**Prerequisites**

The tool that you use to connect to your Linux instance depends on the operating system running on your computer. If your computer runs Windows, you'll connect using PuTTY. If your computer runs Linux or Mac OS X, you'll connect using the SSH client. These tools require the use of your key pair. Be sure that you created your key pair as described in Create a Key Pair.

**To connect to your Linux instance from Windows using PuTTY**

    1.Start PuTTY (from the Start menu, click All Programs > PuTTY > PuTTY).

    2.In the Category pane, select Session and complete the following fields:

        a.In Host Name, enter ec2-user@public_dns_name.

        b.Ensure that Port is 22.
		
	3.In the Category pane, under Connection type, expand SSH, and then select Auth. Complete the following:

		a.Click Browse.

		b.Select the .ppk file that you generated for your key pair, as described in Create a Key Pair, and then click Open.

		c.Click Open to start the PuTTY session.
		
	4.If this is the first time you have connected to this instance, PuTTY displays a security alert dialog box that asks whether you trust the host you are connecting to. Click Yes. A window opens and you are connected to your instance.
	
###Configure the EC2 Instance

**To start the web server**

    1.To ensure that your software packages are up to date on your instance, use the following command to perform a quick software update.
	`[ec2-user ~]$ sudo yum update -y`
	
	2.Install the Apache web server, PHP, and MySQL software packages using the following command.
	`[ec2-user ~]$ sudo yum install -y httpd24 php56 mysql55-server php56-mysqlnd php56-mbstring php56-gd php56-opcache`
	
	3.Start the Apache web server using the following command.
	`[ec2-user ~]$ sudo service httpd start`

	4.Configure the Apache web server to start at each system boot using the following command.
	`[ec2-user ~]$ sudo chkconfig httpd on`

	5.Before you continue, verify that the web server is running. In a web browser on your computer, paste the public DNS name of your instance into the address bar and press Enter. This displays the Apache test page. If you don't see the test page for Apache, verify that your security group allows HTTP traffic.
	
###Setting up the website

	1. Change ownership of /var/www/html to ec2-user
	`[ec2-user ~]$sudo chown -R ec2-user /var/www/html`
	
	2. Move the files of your website into /var/www/html