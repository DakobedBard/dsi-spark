##Miniquiz

We will be needing Amazon AWS services and PySpark for today and tomorrow. Today's miniquiz is
to set those up. If you have troubles, let us know.

##1. Setting up your AWS account
Amazon will ask you for your credit card information during the setup process. You will be charged
for using their services. You should not have to spend more than 5-10 dollars.

Go to [http://aws.amazon.com/](http://aws.amazon.com/) and sign up: 
- You may sign in using your existing Amazon account or you can create a new account by selecting
  "Create a free account" using the button at the right, then selecting "I am a new user."
- Enter your contact information and confirm your acceptance of the AWS Customer Agreement.
- Once you have created an Amazon Web Services Account, you may need to accept a telephone call to
  verify your identity. Some people have used Google Voice successfully if you don't have or don't
  want to give a mobile number.
- Once you have an account, go to [http://aws.amazon.com/](http://aws.amazon.com/) and sign in. You
  will work primarily from the Amazon Management Console.
- Create Security Credentials.  Go to the 
  [AWS security credentials page](https://console.aws.amazon.com/iam/home?#security_credential).
  If you are asked about IAM users, close the message.  Expand "Access Keys" and click "Create New
  Access Key."  You will see a message Your access key (access key ID and secret access key) has been
  created successfully. Click "Download Key File" and make note of where you saved the file.
- Open the Access Key file you saved. Include the following lines in the `~/.bashrc` or `~/.zshrc` file.
  PySpark needs those environment variables to access s3 files stored on Amazon.

  ```
  export AWS_ACCESS_KEY_ID=xxxxxxxxxx
  export AWS_SECRET_ACCESS_KEY=xxxxxxxxxxx
  ```
  
##2. Setting up an EC2 key pair
To connect to the Amazon EC2 instances you will be creating tomorrow, you need to create an SSH key pair. 

- After setting up your account, go to the [EC2 console](https://console.aws.amazon.com/ec2). On the left
  click **Key Pair** and then **Create Key Pair**
  
  <img height="400px" src="images/keypair.png">
    
- Download and save the `.pem` private key file to disk.
  Make sure only you can access the .pem file. If you do not change the permissions,
  you will get an error message at a later step.  Change the permissions using this command:

  ```$ chmod 600 </path/to/saved/keypair/file.pem>```


##3. Spark / PySpark Installation

1. Go to this [link](http://spark.apache.org/downloads.html). 
2. Select `Pre-built for Hadoop 1.X` under `2. Chose a package type:`.
3. Download the tar package by `Download Spark: spark-1.2.0-bin-hadoop1.tgz`
4. Unzip the file and place it at your home directory
5. Make sure the folder name is `spark-1.2.1-bin-hadoop1`
5. Open up your `~/.bashrc` in a text editor or `~/.zshrc` if you are using the z-shell
6. Include the following lines in the `~/.bashrc` or `~/.zshrc` file

   ```
   export SPARK_HOME=[path to your unzipped spark folder]
   export PYTHONPATH=[path to your unzipped spark folder]python/:$PYTHONPATH
   ```

7. Now open up an ipython terminal and `import pyspark as ps`. If it did not throw an error,
   then you are ready to go.
   

