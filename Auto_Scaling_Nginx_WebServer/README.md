# Project 5: Auto-Scaling Nginx Web Application

This project implements a resilient and elastic web application environment using an Auto Scaling Group (ASG) to automatically manage EC2 instances based on CPU load. The system is designed to be self-healing and to scale dynamically to meet traffic demands.

## Architecture

* **Launch Template:** A blueprint defining the EC2 instance configuration, including a user data script to install Nginx.
* **Auto Scaling Group (ASG):** Manages the lifecycle of EC2 instances, ensuring a desired number are always running and scaling in or out based on policies.
* **Application Load Balancer (ALB):** Distributes traffic across the instances in the ASG.
* **CloudWatch Alarms:** Triggers for the scaling policies, based on average CPU utilization.

---

## Step-by-Step Instructions

### 1. Create the Required IAM Role

For connectivity services like Session Manager to work, the EC2 instance needs an IAM role.

1.  Go to the **IAM Dashboard** > **Roles** > **"Create role"**.
2.  **Trusted entity type:** `AWS service`.
3.  **Use case:** `EC2`. Click **Next**.
4.  Search for and attach the policy named **`AmazonSSMManagedInstanceCore`**.
5.  **Role name:** `EC2-SSM-Role`. Create the role.

### 2. Create the Launch Template

1.  In the **EC2 Dashboard**, go to **Launch Templates > Create launch template**.
2.  **Name:** `nginx-template`.
3.  **AMI:** `Amazon Linux 2023 AMI`.
4.  **Instance Type:** `t2.micro`.
5.  **Key Pair:** Select your key pair.
6.  **Network Settings:**
    * **Security groups:** Create a new security group named `nginx-instance-sg`. Add two inbound rules:
        * Allow `HTTP` on port `80` (you will set the source to the ALB's security group later).
        * Allow `SSH` on port `22` from `My IP`.
    * Expand **"Advanced network configuration"**, click "Add network interface," and for **"Auto-assign public IP"**, select **`Enable`**.
7.  **Advanced Details:**
    * **IAM instance profile:** Select the `EC2-SSM-Role` you created.
    * In the **User Data** field, paste the following script:
        ```bash
        #!/bin/bash
        sudo yum update -y
        sudo yum install -y nginx
        sudo systemctl start nginx
        sudo systemctl enable nginx
        ```
8.  Create the launch template.

### 3. Create Networking and Load Balancer Components

1.  **VPC:** Use the **"VPC and more"** wizard to create a new VPC named `nginx-vpc` with 2 AZs, 2 public subnets, 2 private subnets, and 1 NAT Gateway per AZ.
2.  **ALB Security Group:** Create a new security group in `nginx-vpc` named `nginx-alb-sg`. Add one inbound rule to allow `HTTP` on port `80` from `Anywhere (0.0.0.0/0)`.
3.  **Update Instance Security Group:** Edit the `nginx-instance-sg` you created earlier. For the `HTTP` rule, set the **Source** to be your `nginx-alb-sg`.
4.  **Target Group:** Create a new target group named `nginx-tg` for **Instances**, using the `HTTP` protocol on port `80` and associated with your `nginx-vpc`. Do not register any targets.
5.  **Application Load Balancer:**
    * Create a new **Application Load Balancer** named `nginx-alb`.
    * Make it **Internet-facing**.
    * Select your `nginx-vpc` and the two **public subnets**.
    * For Security Groups, select your `nginx-alb-sg`.
    * Create an `HTTP:80` listener that forwards to your `nginx-tg` target group.

### 4. Create the Auto Scaling Group

1.  In the EC2 Dashboard, go to **Auto Scaling Groups > Create Auto Scaling group**.
2.  **Name:** `nginx-asg`.
3.  **Launch Template:** Select your `nginx-template`.
4.  **Network:** Choose your `nginx-vpc` and the two **public subnets**.
5.  **Load Balancing:** Attach to your existing `nginx-tg` and enable ELB health checks.
6.  **Group Size and Scaling:**
    * **Desired/Min/Max Capacity:** `1` / `1` / `3`.
    * **Scaling Policy:** Create a `Target tracking scaling` policy for `Average CPU utilization` with a target value of `50%`.
7.  Create the group.

### 5. Final Verification

* After the ASG launches an instance, check the **Target Groups** page to see its status become `healthy`.
* Navigate to the **DNS name** of the `nginx-alb` in a browser. You should see the **"Welcome to nginx!"** page.

### 6. Cleanup

* To avoid costs, delete all the resources you created in reverse order:
    1.  Auto Scaling Group (this will terminate the EC2 instance).
    2.  Application Load Balancer.
    3.  Launch Template.
    4.  NAT Gateways.
    5.  VPC.
    6.  IAM Role.
