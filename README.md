# AWS Online Movie Application

This project deploys the **Binge Watch Online** movie website on Amazon Web Services (AWS).  
The website is hosted on an Amazon Linux EC2 instance, the application package is stored in Amazon S3, and an Application Load Balancer distributes traffic to EC2 instances.

## Project Description

Binge Watch Online is an online entertainment provider. The company website was initially hosted on a single server, which caused slow page loading and limited scalability.

This project improves the deployment by using AWS cloud services to host the website, distribute traffic, automatically scale during high CPU usage, send monitoring alerts, and track project costs.

## AWS Services Used

- Amazon EC2
- Amazon S3
- AWS IAM
- Application Load Balancer (ALB)
- Target Group
- Launch Template
- Auto Scaling Group
- Amazon CloudWatch
- Amazon SNS
- Amazon Route 53
- AWS Budgets
- Cost Explorer

## Architecture

1. The user accesses the website using the Application Load Balancer DNS name.
2. The Application Load Balancer forwards HTTP traffic on port 80 to the target group.
3. The target group sends traffic to healthy EC2 instances.
4. Each EC2 instance runs Apache HTTP Server and serves the movie website.
5. The Movie.zip application package is stored in Amazon S3.
6. An IAM role allows EC2 to download files from S3.
7. CloudWatch monitors CPU utilization.
8. When CPU utilization exceeds 40%, SNS sends an email alert.
9. The Auto Scaling Group launches additional EC2 instances to handle increased load.
10. Route 53 private hosted zone provides internal DNS access from the Windows VM.

## Deployment Steps

### 1. Launch Linux EC2 Instance

- Open AWS EC2.
- Launch an Amazon Linux instance.
- Select an instance type such as `t2.micro` or `t3.micro`.
- Create and download a key pair.
- Allow SSH port 22 and HTTP port 80 in the security group.
- Enable auto-assign public IP.
- Launch the instance.

### 2. Create S3 Bucket

- Open Amazon S3.
- Create a bucket named `movie-chandan-watch-app-1307`.
- Upload `Movie.zip`.
- Enable public access if required for the project.

### 3. Create IAM Role

- Open IAM.
- Create a role for EC2.
- Attach the `AmazonS3ReadOnlyAccess` policy.
- Attach the role to the EC2 instance.

### 4. Deploy Website on EC2

Connect to EC2:

```bash
ssh -i "movieappkey1.pem" ec2-user@ec2-13-127-252-198.ap-south-1.compute.amazonaws.com
```

Install and start Apache:

```bash
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl status httpd
```

Download and deploy the website:

```bash
sudo chmod 0777 /var/www/html
aws s3 cp s3://movie-chandan-watch-app-1307/Movie.zip ./app.zip
sudo yum install -y unzip
unzip app.zip -d /var/www/html
```

Test the website:

```text
http://13.127.252.198
```

### 5. Create Target Group

- Open EC2 → Target Groups.
- Create a target group.
- Select target type: `Instances`.
- Select protocol: `HTTP`.
- Set port: `80`.
- Register the EC2 instance.
- Confirm the target becomes healthy.

### 6. Create Application Load Balancer

- Open EC2 → Load Balancers.
- Create an Application Load Balancer.
- Select `Internet-facing`.
- Select the same VPC as EC2.
- Select at least two public subnets.
- Create an HTTP listener on port 80.
- Forward traffic to the target group.
- Access the website using the ALB DNS name.

### 7. Create Launch Template and Auto Scaling Group

- Create a launch template.
- Add a user-data script to install Apache and download the website from S3.
- Create an Auto Scaling Group.
- Set minimum capacity: 1.
- Set desired capacity: 1.
- Set maximum capacity: 3.
- Attach the existing target group.
- Configure scaling based on average CPU utilization.

### 8. Create CloudWatch Alarm and SNS Notification

- Create an SNS topic.
- Add an email subscription.
- Confirm the subscription.
- Create a CloudWatch alarm for CPU utilization greater than 40%.
- Select the SNS topic for notifications.

Run the stress test:

```bash
sudo yum install -y amazon-efs-utils
sudo amazon-linux-extras install epel -y
sudo yum install -y stress
stress -c 4
```

When CPU utilization exceeds 40%, CloudWatch triggers the alarm, SNS sends an email, and Auto Scaling launches an additional EC2 instance.

### 9. Create Route 53 Private Hosted Zone

- Open Route 53.
- Create a private hosted zone named `movie.internal`.
- Associate it with the project VPC.
- Create an A record named `www`.
- Map the record to the Application Load Balancer.
- Access the website using:

```text
http://www.movie.internal.movie.internal
```

### 10. Launch Windows VM

- Launch a Windows Server EC2 instance.
- Use the same VPC as the ALB and Linux EC2 instance.
- Allow RDP port 3389.
- Connect using RDP.
- Open the private DNS name in the browser.
- Test the website using the private hosted-zone DNS.

## Billing and Cost Management

- Created an AWS Budget for monthly cost tracking.
- Configured budget alerts at 20%, 40%, 60%, and 80%.
- Added resource tags to EC2 and other project resources.
- Used Cost Explorer to monitor costs by tag.

## Resource Tags

| Key | Value |
|---|---|
| Project | OnlineMovieApplication |
| Environment | Development |
| Owner | Padma |
| CostCenter | BingeWatch |

## Testing

- Website was accessed using the EC2 public IP.
- Website was accessed using the Application Load Balancer DNS.
- EC2 target health was verified.
- CPU stress test was performed.
- CloudWatch alarm and SNS email notification were verified.
- Auto Scaling launched additional EC2 instances.
- Private DNS was tested from the Windows VM.
- AWS Budget and resource tags were configured.

## Important Files

- `Movie.zip` — Movie website application package.
- `commands.txt` — Commands used during the project.
- `screenshots/` — AWS console screenshots.
- `README.md` — Project documentation.

## Security Note

For this academic project, SSH port 22 and HTTP port 80 are open to `0.0.0.0/0`.  
In a production environment, SSH should be restricted to specific IP addresses, and HTTPS should be enabled using AWS Certificate Manager.