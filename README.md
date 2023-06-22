# Assignment 123

Answer que 1
# Define AWS provider
provider "aws" {
  region = "us-east-1"  # Set your desired region
}

# Create VPC
resource "aws_vpc" "example_vpc" {
  cidr_block = "10.0.0.0/16"
}

# Create Subnet
resource "aws_subnet" "example_subnet" {
  vpc_id     = aws_vpc.example_vpc.id
  cidr_block = "10.0.1.0/24"
}

# Create Security Group for EC2 instances
resource "aws_security_group" "example_sg" {
  vpc_id = aws_vpc.example_vpc.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Create EC2 instances
resource "aws_instance" "example_instance1" {
  ami           = "ami-12345678"  # Replace with desired AMI ID
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.example_subnet.id
  security_group_ids = [aws_security_group.example_sg.id]
}

resource "aws_instance" "example_instance2" {
  ami           = "ami-12345678"  # Replace with desired AMI ID
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.example_subnet.id
  security_group_ids = [aws_security_group.example_sg.id]
}

# Create Load Balancer
resource "aws_lb" "example_lb" {
  name               = "example-lb"
  subnets            = [aws_subnet.example_subnet.id]
  security_groups    = [aws_security_group.example_sg.id]
  load_balancer_type = "application"
}

# Attach Instances to Load Balancer
resource "aws_lb_target_group_attachment" "example_attachment1" {
  target_group_arn = aws_lb.example_lb.arn
  target_id        = aws_instance.example_instance1.id
  port             = 80
}

resource "aws_lb_target_group_attachment" "example_attachment2" {
  target_group_arn = aws_lb.example_lb.arn
  target_id        = aws_instance.example_instance2.id
  port             = 80
}

que2.-------------------------------------------------------------------------------------

import boto3

def create_cpu_alarm(instance_id, alarm_name):
    client = boto3.client('cloudwatch')

    # Define the alarm threshold and duration
    threshold_percentage = 80
    evaluation_periods = 5

    # Create the alarm
    response = client.put_metric_alarm(
        AlarmName=alarm_name,
        AlarmDescription='CPU Usage exceeds 80%',
        ActionsEnabled=False,  # Set to True if you want to enable actions (e.g., sending notifications)
        AlarmActions=[],  # Add ARNs of actions (e.g., SNS topics) if ActionsEnabled is True
        MetricName='CPUUtilization',
        Namespace='AWS/EC2',
        Statistic='Average',
        Dimensions=[
            {
                'Name': 'InstanceId',
                'Value': instance_id
            },
        ],
        Period=60,
        EvaluationPeriods=evaluation_periods,
        Threshold=threshold_percentage,
        ComparisonOperator='GreaterThanOrEqualToThreshold'
    )

    print(f"Alarm created successfully: {response['AlarmArn']}")

que3----------------------------------------------------------------------------------------------------

pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        // Checkout source code from version control (e.g., Git)
        git 'https://github.com/your-repo.git'
        
        // Build your application (e.g., compile, package, etc.)
        // You may need to adjust this step based on your application's build process
        sh 'mvn clean package'
      }
    }

    stage('Test') {
      steps {
        // Run tests for your application (e.g., unit tests, integration tests, etc.)
        // You may need to adjust this step based on your testing framework
        sh 'mvn test'
      }
    }

    stage('Deploy') {
      environment {
        // Set AWS credentials as environment variables
        AWS_ACCESS_KEY_ID = credentials('your-aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('your-aws-secret-access-key')
        AWS_REGION = 'us-east-1' // Replace with your desired AWS region
      }
      
      steps {
        // Use AWS CLI to deploy the code to the EC2 instance
        sh 'aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID'
        sh 'aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY'
        sh 'aws configure set region $AWS_REGION'
        sh 'aws s3 cp your-application.jar s3://your-bucket/'
        sh 'aws ec2 run-instances --image-id your-ami-id --instance-type t2.micro --user-data "sudo aws s3 cp s3://your-bucket/your-application.jar /var/www/html/"'
      }
    }
  }
}



