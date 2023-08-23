# Hosting a Dynamic Website on AWS - README

This README document outlines the steps required to host a dynamic website on AWS using various AWS services. The setup will include a 3-tier architecture, load balancer, autoscaling group, and other necessary components to ensure high availability, fault tolerance, scalability, and elasticity.

## AWS Services and Requirements

1. **VPC and 3-tier Architecture**: Create a Virtual Private Cloud (VPC) with the following subnets:
   - Public-Subnet-az1-cidr: 10.0.0.0/24
   - Public-Subnet-az2-cidr: 10.0.1.0/24
   - Private-App-Subnet-az1: 10.0.2.0/24
   - Private-App-Subnet-az2: 10.0.3.0/24
   - Private-Data-Subnet-az1: 10.0.4.0/24
   - Private-Data-Subnet-az2: 10.0.5.0/24

2. **Internet Gateway**: Attach an Internet Gateway to the VPC to enable communication with the internet.

3. **NAT Gateway and Bastion Host**: Set up a NAT Gateway and a Bastion Host for secure access to instances in private subnets.

4. **Application Load Balancer (ALB)**: Create an Application Load Balancer to distribute incoming traffic across multiple EC2 instances.

5. **EC2 Instances**: Launch EC2 instances in private subnets to host the application.

6. **Autoscaling Group**: Set up an Autoscaling Group to maintain high availability, fault tolerance, scalability, and elasticity.

7. **Route 53**: Configure Route 53 for DNS management and routing traffic to the ALB.

8. **Amazon S3**: Create two S3 buckets to store web files: `fleetcart.zip` and `dummy.zip`.

9. **IAM Role**: Create an IAM role with S3 permissions to allow EC2 instances to access and download web files from the S3 buckets.

10. **Certificate Manager**: Use Certificate Manager to obtain TLS certificates for secure communication.

11. **AMI Creation**: Create an AMI from the setup server for future use in launching instances.

12. **Security Groups**: Configure the necessary security groups to control inbound and outbound traffic.

## Procedure

Follow the step-by-step instructions below to set up the dynamic website:

### 1. Update EC2 Instance

```
sudo su
sudo yum update -y
```

### 2. Install Apache

```
sudo yum install -y httpd httpd-tools mod_ssl
sudo systemctl enable httpd
sudo systemctl start httpd
```

### 3. Install PHP 7.4

```
sudo amazon-linux-extras enable php7.4
sudo yum clean metadata
sudo yum install php php-common php-pear -y
sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y
```

### 4. Install MySQL 5.7

```
sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
sudo yum install mysql-community-server -y
sudo systemctl enable mysqld
sudo systemctl start mysqld
```

### 5. Set Permissions

```
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
```

### 6. Download and Unzip FleetCart

```
sudo aws s3 sync s3://fleetcart-web-files-001 /var/www/html
cd /var/www/html
sudo unzip FleetCart.zip
sudo mv FleetCart/* /var/www/html 
sudo mv FleetCart/.* /var/www/html
sudo rm -rf FleetCart FleetCart.zip
```

### 7. Enable Mod Rewrite and Restart Apache

```
sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf
chown apache:apache -R /var/www/html 
sudo service httpd restart
```

### 8. Download and Set Up Dummy Application

```
sudo su
sudo aws s3 sync s3://dummy-web-files-001 .
sudo unzip dummy.zip
sudo mv dummy/* /var/www/html/public
sudo mv -f dummy/.DS_Store /var/www/html/public
sudo rm -rf /var/www/html/storage/framework/cache/data/cache
sudo rm -rf dummy dummy.zip
chown apache:apache -R /var/www/html 
sudo service httpd restart
```


### 8A Migrate the sql script into the rds DB.

### 9. Update .env File

SSH into one of the EC2 instances launched in the private-app subnet. Update the .env file in the /var/www/html directory with your Route 53 domain name (including 'https' but without the trailing '/') and update the APP_URL accordingly.

### 10. Create Bastion Host and SSH to Private Subnet Instances

Before you can SSH into the EC2 instances in the private subnet, create a bastion host and use it to SSH to the EC2 instances in the private subnet. You can also use the `ssh-add --apple-use-keychain key.pem` command to add your key.pem file and then SSH with `-A` option.

### 11. Create AMI for Autoscaling Group

Once your application is working correctly, create an AMI out of the EC2 instance in the private subnet. This AMI will be used to create an Autoscaling Group (ASG) for your application to add high availability, scalability, fault tolerance, and elasticity.

## Conclusion

Following these instructions will help you set up a dynamic website on AWS using a variety of AWS services, providing you with valuable hands-on experience and knowledge of core AWS services and how they work together to host a web application.
