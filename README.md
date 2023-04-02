# Build-BI-System-from-Scratch

The purpose of this lab is to implement a Businesss Intelligence(BI) System using AWS Analytics Services.
will set up a Data ``Collection -> Store -> Analysis/Processing -> Visualization`` pipeline.

we are going to implement the following Businesss Intelligence(BI) System using AWS Analytics Services.

Amazon Kinesis Data Streams and Kinesis Data Firehose are used for data collection. Amazon S3 is for data storage. AWS Lambda, Amazon Athena, and Amazon Elasticsearch Service are for data analysis and processing. Amazon QuickSight and Kibana are for Visualization.

## Creating an IAM User

- Create an IAM user and allow ``AdministratorAccess`` permissions.
- Make sure to choose both ``Programmatic access`` and ``AWS Management Console access``.
- Download the new user's credentials.

## Creating Security Groups

### Security Group for EC2 instance as a bastion host
We will create a security group to use with an Amazon EC2 instance which will serve as a bastion host.

**Basic details**
- Security group name : ``bastion``
- Description : ``security group for bastion``

**Inbound rules**
- Type : SSH
- Protocol : TCP
- Port Range : 22
- Source : ``Your network's public IPv4 address range``

### Security Group for use with Elasticsearch Service
**Basic Details**
- Security group name : ``use-es-cluster-sg``
- Description : ``security group for an es client``

**Inbound rules** Leave empty.

### A other Security Group 
**Basic details**
- Security group name : ``es-cluster-sg``
- Description : ``security group for an es cluster``

**Inbound rules**
- Type : All TCP
- Protocol : TCP
- Port Range : 0-65535
- Source : ``use-es-cluster-sg`` from the dropdown list. For example: sg-038b632ef1825cb7f
**Outbound rules** Leave at default values.

## Launch an EC2 Instance
We will create an EC2 instance that will run code to generate the data needed for the lab.

- Choose an Amazon Machine Image (AMI).
- Choose Amazon Linux 2 AMI (HVM), SSD Volume Type.
- Select t2.micro as the instance type
- Select an existing security group from Assign a security group, and select bastion and use-es-cluster-sg from the Security Group table.
- Create a key pair to access EC2 Instance.

## Configuring your EC2 Instance
In this step we will configure the EC2 instance to access and control other AWS resources.

Enter the following commands once you are logged into the EC2 instance:
(1) Download the source code.
```
wget 'https://github.com/aws-samples/aws-analytics-immersion-day/archive/refs/heads/main.zip'
```

(2) Extract the downloaded source code.
```
unzip -u main.zip
```
(3) Grant execution authority to the practice environment setting script.
````
chmod +x ./aws-analytics-immersion-day-main/set-up-hands-on-lab.sh
```
(4) Execute the setup script to set the lab environment.
```
./aws-analytics-immersion-day-main/set-up-hands-on-lab.sh
```
(5) Make sure the files necessary for the lab are normally created after running the configuration script. For example, check if the source code and necessary files exist as shown below.
```
[ec2-user@ip-172-31-2-252 ~]$ ls -1
athena_ctas.py
aws-analytics-immersion-day-main
gen_kinesis_data.py
main.zip
upsert_to_es.py
````
In order to run the Python synthentic data generator script (``gen_kinesis_data.py``), we need to set user credentials by following the instructions:

Perform aws configure to access other AWS resources. At this time, the IAM User data created earlier is used. Open the previously downloaded .csv file, check the Access key ID and Secret access key, and enter them.

```
$ aws configure
AWS Access Key ID [None]: <Access key ID>
AWS Secret Access Key [None]: <Secret access key>
Default region name [None]: us-east-1
Default output format [None]:
```
If the setting is complete, the information entered as follows will be masked.
```
$ aws configure
AWS Access Key ID [****************EETA]:
AWS Secret Access Key [****************CixY]:
Default region name [None]: us-east-1
Default output format [None]:
```
## Build up Data Analytics System
We are now ready to build a data analysis system using the AWS Analytics services. 

### Create Kinesis Data Stream to receive input dataHeader anchor link
- Select Kinesis from the list of services on the AWS Management Console.
- Make sure the Kinesis Data Streams radio button is selected and click Create data stream button.
- Enter retail-trans as the Data stream name.
- Choose either the On-demand or Provisioned capacity mode. With the On-demand mode, you can then choose Create Kinesis stream to create your data stream. With the Provisioned mode, you must then specify the number of shards you need, and then choose Create Kinesis stream. If you choose Provisioned mode, enter 1 in Number of open shards under Data stream capacity.
- Click the Create data stream button and wait for the status of the created kinesis stream to become active.

### Create Kinesis Data Firehose to store data in S3Header anchor link
Kinesis Data Firehose will allow collecting data in real-time and batch it to load into a storage location such as Amazon S3, Amazon Redshift or ElasticSearch.

