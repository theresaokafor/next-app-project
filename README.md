# next-app-project
this project seeks to deploy a dynamic web application on aws cloud.

# Hosting a Dynamic Web Application on EC2 and AWS Cloud
This README provides step-by-step instructions for hosting a dynamic web application on Amazon EC2 and AWS Cloud. Follow these steps to successfully deploy your application.

## Task Steps
1. **Create Buckets**: Create two S3 buckets to store application codes and SQL data.
2. **Create a 3-Tier VPC**: Set up a three-tier Virtual Private Cloud (VPC) to isolate your resources.
3. **Create Subnet Group and RDS DB**: Create a subnet group and an Amazon RDS database for your application.
4. **Create Security Groups**: Create security groups for ALB, webserver, RDS, EIC instance, and EIC endpoint.
5. **Create EIC Endpoint**: Establish an Elastic Instance Connect endpoint to access tthe private subnets for updates.
6. **Launch EC2 Instance**: Launch an EC2 instance (webserver) and migrate SQL data from S3 to the RDS database using Flyway.

    ```bash
    # Install Flyway
    wget -qO- https://download.red-gate.com/maven/release/org/flywaydb/enterprise/flyway-commandline/9.21.1/flyway-commandline-9.21.1-linux-x64.tar.gz | tar -xvz && sudo ln -s `pwd`/flyway-9.21.1/flyway /usr/local/bin
    cd flyway
    aws s3 cp s3://nextapp-sqldata-tess/V1__nest.sql /home/ec2-user/flyway-9.21.1/sql
    flyway -url=jdbc:mysql://<db-endpoint>:3306/<db-name> -user=<db-username> -password=<db-password> -locations=filesystem:sql migrate
    ```

7. **Install LAMP Stack**: Install Apache, PHP 8, and MySQL 8 on the EC2 instance.

    ```bash
    # Update EC2 instance
    sudo yum update -y

    # Install Apache
    sudo yum install -y httpd httpd-tools mod_ssl
    sudo systemctl enable httpd
    sudo systemctl start httpd

    # Install PHP 8
    sudo yum install -y amazon-linux-extras
    sudo amazon-linux-extras enable php8.0
    sudo yum clean metadata
    sudo yum install -y php-cli php-fpm php-mysqlnd php-bcmath php-ctype php-fileinfo php-json php-mbstring php-openssl php-pdo php-gd php-tokenizer php-xml
    php -v

    # Install MySQL 8
    sudo amazon-linux-extras install epel -y
    wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
    dnf install -y mysql80-community-release-el9-1.noarch.rpm
    dnf repolist enabled | grep "mysql.-community."
    dnf install -y mysql-community-server
    sudo systemctl enable mysqld
    sudo systemctl start mysqld

    # Enable PHP Curl Module
    sudo yum install php-curl
    sudo systemctl restart httpd

    # Update php.ini settings
    sudo sed -i 's/^max_execution_time =.*/max_execution_time = 300/' /etc/php.ini
    sudo sed -i 's/^memory_limit =.*/memory_limit = 128M/' /etc/php.ini
    sudo systemctl restart httpd

    # Enable mod_rewrite module
    sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf
    sudo systemctl restart httpd
    ```

8. **Copy App Code**: Copy and move the contents of the app code from S3 into `/var/www/html`.

    ```bash
    aws s3 cp s3://nextapp-tess/nest-app.zip /var/www/html
    cd /var/www/html
    sudo unzip nest-app.zip
    sudo mv nest-app/* /var/www/html
    sudo mv nest-app/.* /var/www/html
    sudo rm -rf nest-app nest-app.zip
    ```

9. **Set Permissions**: Set permissions `777` for the `/var/www/html` directory and the `storage/` directory.

    ```bash
    sudo chmod -R 777 /var/www/html
    sudo chmod -R 777 /var/www/html/storage
    ```

10. **Edit `.env` File**: Edit the `.env` file in the `html` directory and add database details.

    ```bash
    sudo vi /var/www/html/.env
    ```

11. **Edit `AppServiceProvider.php

`**: Open the 'AppServiceProvider.php' file in the '/var/www/html/app/Providers' directory and add the public function boot code.

    ```bash
    sudo vi /var/www/html/app/Providers/AppServiceProvider.php
    sudo service httpd restart
    ```

12. **Create an AMI**: Create an Amazon Machine Image (AMI) using the web server where installation was done.
13. **Create Target Group & ALB**: Create a Target Group and an Application Load Balancer (ALB).
14. **Register Domain Name**: Register a domain name and create a record. Register an SSL certificate for secure communication and create an HTTPS listener.
15. **Create Launch Template & Auto Scaling Group**: Create a Launch Template and an Auto Scaling Group using the previously created AMI.
16. **Check Website**: Verify that the website is up and running.
