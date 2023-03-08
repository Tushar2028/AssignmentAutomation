#!/bin/bash

# define variables
myname="aadarsh"
s3_bucket="upgrad-aadarsh"

# update package details and package list
sudo apt update -y

# check if apache2 package is installed
if dpkg -s apache2 2>/dev/null | grep -q "Status: install ok installed"; then
    echo "Apache2 package is already installed"
else
    # install apache2 package if not already installed
    echo "Installing Apache2 package..."
    sudo apt install apache2 -y
    if [ $? -eq 0 ]; then
        echo "Apache2 package installed successfully"
    else
        echo "Error installing Apache2 package"
        exit 1
    fi
fi

# check if apache2 service is running
if systemctl is-active --quiet apache2; then
    echo "Apache2 service is running"
else
    # start apache2 service if not already running
    echo "Starting Apache2 service..."
    sudo systemctl start apache2
    if [ $? -eq 0 ]; then
        echo "Apache2 service started successfully"
    else
        echo "Error starting Apache2 service"
        exit 1
    fi
fi

# check if apache2 service is enabled
if systemctl is-enabled --quiet apache2; then
    echo "Apache2 service is enabled"
else
    # enable apache2 service if not already enabled
    echo "Enabling Apache2 service..."
    sudo systemctl enable apache2
    if [ $? -eq 0 ]; then
        echo "Apache2 service enabled successfully"
    else
        echo "Error enabling Apache2 service"
        exit 1
    fi
fi

# check if /var/log/apache2/ directory exists
if [ ! -d "/var/log/apache2/" ]; then
    echo "/var/log/apache2/ directory not found"
    exit 1
fi

# create timestamp variable
timestamp=$(date '+%d%m%Y-%H%M%S')

# create tar archive of apache2 access and error logs in /var/log/apache2/ directory
# only include .log files, not other file types like .zip or .tar
echo "Creating tar archive of Apache2 logs..."
sudo tar -cvzf /tmp/${myname}-httpd-logs-${timestamp}.tar /var/log/apache2/*.log
if [ $? -eq 0 ]; then
    echo "Tar archive created successfully: /tmp/${myname}-httpd-logs-${timestamp}.tar"
else
    echo "Error creating tar archive"
    exit 1
fi

# check if AWS CLI is installed
if command -v aws > /dev/null; then
    echo "AWS CLI is installed"
else
    # install AWS CLI if not already installed
    echo "Installing AWS CLI..."
    sudo apt install awscli -y
    if [ $? -eq 0 ]; then
        echo "AWS CLI installed successfully"
    else
        echo "Error installing AWS CLI"
        exit 1
    fi
fi

#above code will install aw cli for our next prcoess for copying tar archive to s3 bucket

# copy tar archive to s3 bucket
echo "Copying tar archive to S3 bucket..."
aws s3 cp /tmp/${myname}-httpd-logs-${timestamp}.tar s3://${s3_bucket}/${myname}-httpd-logs-${timestamp}.tar
if [ $? -eq 0 ]; then
   echo "Tar archive copied to S3 bucket successfully: s3://${s3_bucket}/${myname}-httpd-logs-${timestamp}.tar"
else
    echo "Error copying tar archive to S3 bucket"
    exit 1
fi

#first task done

# check for inventory.html file in /var/www/html/ directory
if [ ! -f "/var/www/html/inventory.html" ]; then
    # create inventory.html file if not found
    echo "Creating inventory.html file..."
    printf "Log Type\tTime Created\tType\tSize\n" | sudo tee /var/www/html/inventory.html
    if [ $? -eq 0 ]; then
        echo "Inventory.html file created successfully"
    else
        echo "Error creating inventory.html file"
        exit 1
    fi
fi

# append new entry to inventory.html file
echo "Appending new entry to inventory.html file..."
printf "httpd-logs\t%s\ttar\t%s\n" "${timestamp}" "$(du -h /tmp/${myname}-httpd-logs-${timestamp}.tar | awk '{print $1}')" | sudo tee -a /var/www/html/inventory.html
if [ $? -eq 0 ]; then
    echo "New entry appended to inventory.html file successfully"
else
    echo "Error appending new entry to inventory.html file"
    exit 1
fi

# create a cron job file for the root user
sudo crontab -u root -e

# add the following line at the end of the file to run the script every min
* * * * * /root/Automation_Project/automation.sh
