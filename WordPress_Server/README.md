# Project 1: Basic WordPress Server on EC2

This project demonstrates how to deploy a standard WordPress application on a single Amazon EC2 instance using Amazon Linux 2023. The web server and database are both hosted on the same instance.

## Architecture

* **EC2 Instance:** A `t2.micro` instance running Amazon Linux 2023.
* **Web Server:** Apache (`httpd`).
* **Database:** MariaDB (a community-developed fork of MySQL).
* **Application:** WordPress.
* **Networking:** A default VPC with a security group allowing HTTP and SSH traffic.

## Step-by-Step Instructions

### 1. Launch the EC2 Instance
1.  Navigate to the EC2 Dashboard and click **"Launch instances"**.
2.  **Name:** `WordPress-Server`.
3.  **AMI:** Select the `Amazon Linux 2023 AMI`.
4.  **Instance Type:** `t2.micro`.
5.  **Key Pair:** Create or select an existing key pair.
6.  **Security Group:** Create a new security group with inbound rules allowing `SSH` (Port 22) and `HTTP` (Port 80) from `Anywhere` (`0.0.0.0/0`).
7.  Launch the instance.

### 2. Connect to the Instance and Install Software
1.  Connect to the instance via SSH using its public IP address.
2.  Update packages and install the necessary software:
    ```bash
    sudo yum update -y
    sudo yum install -y httpd php php-mysqlnd mariadb105-server
    ```
3.  Start and enable the services:
    ```bash
    sudo systemctl start httpd
    sudo systemctl enable httpd
    sudo systemctl start mariadb
    sudo systemctl enable mariadb
    ```

### 3. Set Up the Database
1.  Run the secure installation script for MariaDB:
    ```bash
    sudo mysql_secure_installation
    ```
    * Follow the prompts to set a root password and secure the installation.
2.  Log in to the database and create a database and user for WordPress:
    ```bash
    sudo mysql -u root -p
    ```
    * Inside the MySQL prompt, run:
        ```sql
        CREATE DATABASE wordpress_db;
        CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'YourSecurePassword';
        GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress_user'@'localhost';
        FLUSH PRIVILEGES;
        EXIT;
        ```

### 4. Install and Configure WordPress
1.  Download and extract the WordPress files in your home directory:
    ```bash
    wget [https://wordpress.org/latest.tar.gz](https://wordpress.org/latest.tar.gz)
    tar -xzf latest.tar.gz
    ```
2.  Create and edit the `wp-config.php` file:
    ```bash
    cd wordpress/
    cp wp-config-sample.php wp-config.php
    nano wp-config.php
    ```
    * Update the `DB_NAME`, `DB_USER`, and `DB_PASSWORD` with the values you created.
3.  Copy the WordPress files to the web server's root directory and set permissions:
    ```bash
    sudo cp -r * /var/www/html/
    sudo chown -R apache:apache /var/www/html/
    ```
4.  Restart the Apache server:
    ```bash
    sudo systemctl restart httpd
    ```

### 5. Final Verification
* Open a web browser and navigate to the public IP address of your EC2 instance. You should see the WordPress installation screen.
* eg . http://ec2-instance-ipv4-address
