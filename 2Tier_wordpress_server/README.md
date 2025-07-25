# Project 4: Advanced WordPress Architecture

This project builds a secure, scalable, and highly available two-tier architecture for a WordPress application. This design follows AWS best practices by isolating resources in private subnets and using managed services.

## Architecture

* **VPC:** A custom VPC with public and private subnets across two Availability Zones.
* **Application Load Balancer (ALB):** Placed in the public subnets to distribute traffic.
* **EC2 Instance:** A `t2.micro` running WordPress in a private subnet, inaccessible from the internet.
* **RDS Database:** A `db.t3.micro` managed MySQL instance in a private subnet.
* **NAT Gateway:** Allows the private EC2 instance to access the internet for updates.
* **Security Groups:** A multi-layered security group configuration to control traffic between tiers.

---

## Step-by-Step Instructions

### 1. Create the VPC Network Foundation

1.  In the VPC Dashboard, use the **"VPC and more"** wizard to create a new VPC.
2.  **Configuration:**
    * **Name:** `wordpress-prod-vpc`.
    * **Availability Zones:** 2.
    * **Subnets:** 2 public and 2 private.
    * **NAT gateways:** 1 per AZ.
3.  After creation, select your new VPC, go to **"Actions" > "Edit VPC settings"**, and ensure **"Enable DNS hostnames"** is checked. Save the changes.

### 2. Create Security Groups

1.  **ALB Security Group (`alb-sg`):** Create a security group in your new VPC that allows inbound `HTTP` traffic on port 80 from `Anywhere (0.0.0.0/0)`.
2.  **Web Server Security Group (`web-sg`):** Create a security group in your VPC that allows:
    * Inbound `HTTP` traffic on port 80 from the `alb-sg` you just created.
    * Inbound `SSH` traffic on port 22 from your personal IP address (`My IP`).
3.  **Database Security Group (`db-sg`):** Create a security group in your VPC that allows inbound `MYSQL/Aurora` traffic on port 3306 from the `web-sg`.

### 3. Create the RDS Database Tier

1.  In the RDS Dashboard, click **"Create database"**.
2.  **Configuration:**
    * **Engine:** `MySQL`.
    * **Template:** `Free tier`.
    * **Settings:**
        * **DB instance identifier:** `wordpress-db`.
        * **Master username:** `dbadmin`.
        * **Master password:** Create and save a secure password.
    * **Initial database name:** `wordpress_db`.
    * **Connectivity:**
        * **VPC:** Select your `wordpress-prod-vpc`.
        * **Public access:** Select **`No`**.
        * **VPC security group:** Remove the `default` group and select your `db-sg`.
3.  Create the database and wait for its status to become "Available".

### 4. Launch the EC2 Web Server

1.  Launch a `t2.micro` EC2 instance with the `Amazon Linux 2023` AMI.
2.  **Network Settings:**
    * **VPC:** `wordpress-prod-vpc`.
    * **Subnet:** Select a **private subnet**.
    * **Auto-assign Public IP:** `Disable`.
    * **Security Group:** Attach your `web-sg`.
3.  Connect to the instance using **EC2 Instance Connect** or **Session Manager**.
4.  Install Apache and PHP:

    ```bash
    sudo yum update -y
    sudo yum install -y httpd php php-mysqlnd
    sudo systemctl start httpd
    sudo systemctl enable httpd
    ```
### 5. Configure WordPress

1.  In your EC2 terminal, download and extract WordPress.

    ```bash
    wget [https://wordpress.org/latest.tar.gz](https://wordpress.org/latest.tar.gz)
    tar -xzf latest.tar.gz
    ```
2.  Create and edit the `wp-config.php` file.

    ```bash
    cd wordpress/
    cp wp-config-sample.php wp-config.php
    nano wp-config.php
    ```
3.  Update the file with your RDS database details. The `DB_HOST` is the **Endpoint** from your RDS database page.

    ```php
    define( 'DB_NAME', 'wordpress_db' );
    define( 'DB_USER', 'dbadmin' );
    define( 'DB_PASSWORD', 'Your-RDS-Password-Here' );
    define( 'DB_HOST', 'Your-RDS-Endpoint-Here' );
    ```
4.  Copy the configured WordPress files to the web server directory and set permissions.

    ```bash
    sudo cp -r * /var/www/html/
    sudo chown -R apache:apache /var/www/html/
    ```
### 6. Deploy the Application Load Balancer

1.  Create a **Target Group** (`wordpress-tg`) for instances, pointing to your `wordpress-prod-vpc`.
    * **Health Checks:** Expand "Advanced health check settings". Change the **Path** to `/license.txt`. This makes the health check more reliable.
2.  Register your private EC2 instance as a target in the target group.
3.  Create an **Application Load Balancer**.
    * **Scheme:** `Internet-facing`.
    * **VPC:** `wordpress-prod-vpc`.
    * **Mappings:** Select the two **public subnets**.
    * **Security Group:** Attach your `alb-sg`.
    * **Listener:** Create an `HTTP:80` listener that forwards traffic to your `wordpress-tg`.

### 7. Final Verification

* Navigate to the **DNS name** of the Application Load Balancer in a browser. After a few minutes, you should see the WordPress installation screen.

### 8. Cleanup

* To avoid costs, delete all the resources you created in reverse order:
    1.  Application Load Balancer
    2.  EC2 Instance
    3.  RDS Database (choose not to create a final snapshot).
    4.  NAT Gateways
    5.  VPC (This will also delete associated subnets, route tables, etc.).
