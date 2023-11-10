# Creating-a-Website-on-AWS-S3
In this task I am going to launch a static web site using AWS storage service S3. I am going to explain step by step process and AWS console Bahs script.

Below are agenda steps of this task :
* We will create an Amazon Simple Storage Service (Amazon S3) bucket.
* We will create a new AWS Identity and Access Management (IAM) user that has full access to the Amazon S3 service.
* We will upload files to Amazon S3 to host a simple website for the Café & Bakery. And last but not least
* We will create a batch file that can be used to update the static website when you change any of the website files locally.

  ![image](https://github.com/fazalUllah-Khan/Creating-a-Website-on-AWS-S3/assets/148821704/784dad46-4ef7-4eb0-a7d5-605ac529284e)

Note: _Clients will be able to access the website deployed to Amazon S3. The website URL is similar to this example: http://.s3-website-us-west-2.amazonaws.com. Also that created AWS bucket can be access through the AWS Management Console or the AWS CLI._ 

# Step 1 : Accessing the AWS Management Console

Open AWS management console with you user name and password.  
  
_Tip: Use suitable region or do not change the deafault Region._

# Step 2 : Create EC2(OS Linux) and login to it via SSH 

1.	In the AWS Management Console on the Services menu, choose EC2.
2.	In the left navigation pane, choose EC2 dashboard to ensure that you are on the dashboard page.
3.	Choose Launch instance, and then select Launch instance.
4.	In the Name and tags pane, in the Name text box, enter _Web Server_
5.	Choose AMI(Amazon Machine Image), notice that the Amazon Linux 2 AMI image is selected by default. we kept asa.
6.	Select a t3.micro instance. This instance type has 2 virtual CPU and 1 GiB of memory.
7.	In the Key pair (login) pane, select Proceed without a key pair (Not recommended).
8.	In Network settings we confirgured VPC (VPCLAB),and Security group name - required: Web Server security group Description: Security group for my web server Under 
   Inbound security groups rules select the REMOVE.
9. Amazon EC2 stores data on a network-attached virtual disk called Amazon Elastic Block Store (Amazon EBS). We launch the EC2 instance using a default 8 GiB disk volume and in
   Configure storage pane, kept the default storage configuration
10. In advanced settings termination protection enabled. and in user data text box following code is used
_Script for install apache Webserver,cofigured to auto restart,activated,and create simple web page_
 ***
  _#!/bin/bash
  yum -y install httpd
  systemctl enable httpd
  systemctl start httpd
  echo '< html >< h1 >Hello From Your Web Server!</h1></html>' > /var/www/html/index.html_
  ***
Finally our EC2 instance is ready to launch. You can verify instance state "RUNNING" and status test "2/2 checks PASSED" in main console pannel. 

![image](https://github.com/fazalUllah-Khan/Creating-a-Website-on-AWS-S3/assets/148821704/98b8cbd0-d7e0-4792-9220-ced9f2c43779)
 
Now its is time to access EC2 via SSH for which we require and already created PPK file and will use in to login to EC2. 

11.	Check in EC2 information from console pannel and Copy and paste the PublicIP into a text editor which we are gona use to connect to EC2.  
12.	Here we will used putty to SSH EC2 and before that we will do some settings for putty like Configure the PuTTY timeout to keep the PuTTY session open for a longer period of time:
    --> Choose Connection --> For Seconds between keepalives, enter 30 --> Configure your PuTTY session --> Choose Session --> For the Host Name (or IP address), enter the PublicIP 
        address that you copied from the previous steps --> In PuTTY in the Connection list, choose SSH to expand it --> Choose Auth, but don't expand it --> Choose Browse --> Browse to and select the labuser.ppk file that you downloaded --> To choose the file, choose Open--> Choose Open again.
13.	In the PuTTY Security Alert window, choose Accept to trust and connect to the host.
14.	When prompted with login as, enter ec2-user and press Enter.

This step connects you to the EC2 instance.

# Step 3 : Create S3 bucket using AWS CLI. 

Now we created create a new S3 bucket. Bucket name is myWebserver, and it is created in us-west-2 region 

_aws s3api create-bucket --bucket <twhitlock256> --region us-west-2 --create-bucket-configuration LocationConstraint=us-west-2_

Upon succesful creation we can verify JSON-formatted response with a Location name-value pair, where the value reflects the bucket name.

_{
    "Location": "http://twhitlock256.s3.amazonaws.com/"
}_

The command and O/p looks like this in my case

<img width="925" alt="image" src="https://github.com/fazalUllah-Khan/Creating-a-Website-on-AWS-S3/assets/148821704/98cbf6c3-3b47-4a3e-97ee-ee99541fe427">

# Step 4: Create a new IAM user that has full access to Amazon S3

Now some security for S3 access by creating an IAM user. Using the AWS CLI, create a new IAM user with the command aws iam create-user and username awsS3user

_aws iam create-user --user-name awsS3user_ 

As well, Created a login profile for the new user by using the following command:

_aws iam create-login-profile --user-name awsS3user --password Training123!_

O/p and command run my case looks like this :

<img width="670" alt="image" src="https://github.com/fazalUllah-Khan/Creating-a-Website-on-AWS-S3/assets/148821704/8ad6632c-4017-4732-9392-0d165c6d9a6d">

AWS managed policy that grants full access to Amazon S3, run the following command:

_aws iam list-policies --query "Policies[?contains(PolicyName,'S3')]"_

<img width="668" alt="image" src="https://github.com/fazalUllah-Khan/Creating-a-Website-on-AWS-S3/assets/148821704/41cf26d3-3d0f-420f-b3e6-3b5ef669e0c8">

To grant the awsS3user user full access to the S3 bucket, replace <policyYouFound> in following command with the appropriate PolicyName from the results, and run the adjusted command:

_aws iam attach-user-policy --policy-arn arn:aws:iam::aws:policy/<policyYouFound> --user-name awsS3user_

# Step 5: Extract the files for Web page

In the SSH terminal, extract the files we need by running the following commands:

_cd ~/sysops-activity-files_
_tar xvzf static-website-v2.tar.gz_
_cd static-website_

P.S To confirm files extracted , run the _ls_ command. You should see a file named index.html and directories named css and images.

# Step 6: Upload files to Amazon S3 by using the AWS CLI

Upload extracted contents to Amazon S3.	So that the bucket can function as a website, replace <my-bucket> in the following command with your bucket name, and run the adjusted command

_aws s3 website s3://<my-bucket>/ --index-document index.html_

upload the files to the bucket, replace <my-bucket> in the following command 

_aws s3 cp /home/ec2-user/sysops-activity-files/static-website/ s3://<my-bucket>/ --recursive --acl public-read_

verify the files were uploaded, replace <my-bucket> in the following command 

_aws s3 ls <my-bucket>_

On AWS Management Console, on the Amazon S3 console, choose your bucket name. Choose the Properties tab. At the bottom of the this tab, note that Static website hosting is Enabled. Running the aws s3 website AWS CLI command turns on the static website hosting for an Amazon S3 bucket. This option is usually turned off by default. To open the URL on a new page, choose the Bucket website endpoint URL that displays.

# _Congratulations, you have created a static website that is available to the public for viewing!_

# Step 7 : Create a batch file to make updating the website repeatable
Now the last step to create a repeatable deployment, to create a batch file by using the VI editor
So lets first pull up the history of recent commands, 

_history_

Locate the line where you ran the aws s3 cp command. You will put this line in your new batch file

Change directories and create an empty file, run the following command in the SSH terminal session:

_cd ~_
_touch update-website.sh_

open the empty file in the VI editor

_vi update-website.sh_

enter edit mode in the VI editor, press i. And add the standard first line of a bash file and then add the s3 cp line from your history. To do so, replace <my-bucket> in the following command with your bucket name, and copy and paste the adjusted command into your file:

_#!/bin/bash_
_aws s3 cp /home/ec2-user/sysops-activity-files/static-website/ s3://<my-bucket>/ --recursive --acl public-read_

write the changes and quit the file, press Esc, enter :wq and then press Enter.

make the file an executable batch file

_chmod +x update-website.sh_

open the local copy of the index.html file in a text editor, run the following command:

_vi sysops-activity-files/static-website/index.html_

We enter edit mode in the VI editor, press i and modify the file like located HTML code bgcolor="aquamarine" and change this code to bgcolor="gainsboro"; Located the line that has the HTML code bgcolor="orange" and change this code to bgcolor="cornsilk"; Located the second line that has the HTML code bgcolor="aquamarine" and change this code to bgcolor="gainsboro". To write the changes and quit the file  , press Esc, enter :wq and then press Enter.

finally , update the website, run your batch file 
/update-website.sh

P.S _Note: The command line output should show that the files were copied to Amazon S3_

So in end , to see the changes to the website, return to the browser and refresh the Café and Bakery page.

# _Congratulations, you just made your first revision to the website!_









