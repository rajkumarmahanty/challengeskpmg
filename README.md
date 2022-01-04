# challengeskpmg
Challenges from KMPG Interview

What it does

    Query the metadata of an ec2 instance within AWS and provide a json formatted output.
    Retrieve the value of a particular data key.

How to install

    Create an EC2 Linux instance on AWS ( I have used the Ubuntu AMI version 20.x )
    SSH into the instance
    Install Python 3 and git on your instance
        sudo yum install python3 git
    Clone this repository
        git clone https://github.com/bluprince13/aws-metadata-json
    Install pipenv
        sudo pip3 install pipenv
    Open the repository on your instance
        cd aws-metadata-json
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
