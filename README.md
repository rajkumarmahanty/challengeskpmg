# challengeskpmg
Challenges from KMPG Interview

Challenge 2 - We need to write code that will query the meta data of an instance within AWS and provide a json formatted output. The choice of language and implementation is up to you. Bonus Points The code allows for a particular data key to be retrieved individually

What it does

    1. Query the metadata of an ec2 instance within AWS and provide a json formatted output.
    2. Retrieve the value of a particular data key.

How to install

    Create an EC2 Linux instance on AWS ( I have used the Ubuntu AMI version 20.x )
    
    SSH into the EC2instance
    ssh -i "xxx.pem" ubuntu@<<public IP4 DNS of the EC2>>
    
    Install Python3 and PIP3  
    
    sudo apt update
    sudo apt install software-properties-common
    sudo add-apt-repository ppa:deadsnakes/ppa
    sudo apt install python3.8
    python3 --version  ( Checking if python is installed correctly )
    sudo apt install python3-pip  
    pip3 --version ( Checking if python is installed correctly )
    
    cd challenge2/ec2-metadata-json/   ( create and change to your desired directory in the EC2 )
    
    Clone this repository
        git clone https://github.com/rajkumarmahanty/challengeskpmg.git
    
    Open the repository on your instance
        cd ec2-metadata-json
    Install project dependancies
        pipenv install

How to run

    Open the src folder
        cd ec2-metadata-json/src
    
    Run whichever script you need:
    
    To get all the Metadata Values in JSON
        python3 get_fullmetadata.py
        
    To get a particular data key
        python3 get_particularkey.py

How it works

It makes use of the http://169.254.169.254/latest/meta-data link-local address. Instance metatada is provided at this link, but only when you visit it from a running instance.
A few simple Python scripts are used to extract the required information using the above API.

