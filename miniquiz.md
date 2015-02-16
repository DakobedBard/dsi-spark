##Miniquiz

We will be needing Amazon AWS services and PySpark for today and tomorrow. Today's miniquiz is
to set those up. If you have troubles, let us know.

##1. Setting up an AWS account

Go to [http://aws.amazon.com/](http://aws.amazon.com/) and sign up: 

You may sign in using your existing Amazon account or you can create a new account by selecting 
"Create a free account" using the button at the right, then selecting "I am a new user."
Enter your contact information and confirm your acceptance of the AWS Customer Agreement.
Once you have created an Amazon Web Services Account, you may need to accept a telephone call to 
verify your identity. 

Once you have an account, go to [http://aws.amazon.com/](http://aws.amazon.com/) and sign in. 
You will work primarily from the Amazon Management Console. 

**Create Security Credentials**
- Go to the AWS security credentials page 
- If you are asked about IAM users, close the message. Expand "Access Keys" and click "Create New Access Key"
- You will see a message Your access key (access key ID and secret access key) has been created successfully
- Click "Download Key File" and make note of where you saved the file 
- Setting up an EC2 key pair
- To connect to the Amazon EC2 instances you will be creating, you need to create an SSH key pair. 

After setting up your account, follow Amazon's instructions to create a key pair.
Follow the instructions in section "Creating Your Key Pair Using Amazon EC2."
(We have reports that Internet Explorer could make it impossible to download the .pem private key file;
you may want to use a different browser.) Download and save the .pem private key file to disk. 
We will reference the .pem file as `</path/to/saved/keypair/file.pem>` in the following instructions.
Make sure only you can access the .pem file. If you do not change the permissions, you will get an error 
message at a later step.  Change the permissions using this command: `$ chmod 600 </path/to/saved/keypair/file.pem>`
Note: This step will NOT work on Windows 7 with cygwin. Windows 7 does not allow file permissions to be 
changed through this mechanism, and they must be changed for ssh to work. So if you must use Windows, 
you should use PuTTY as your ssh client. In this case, you will further have to transform this key file into 
PuTTY format. For more information go to http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html and look 
under "Private Key Format."
   

##2. Spark / PySpark Installation

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
7. If you have an AWS account, also include the following lines in the `~/.bashrc` or `~/.zshrc` file.
   If you have not got an AWS account, please follow instructions in **Section 2**

   ```
   export AWS_ACCESS_KEY_ID=xxxxxxxxxx
   export AWS_SECRET_ACCESS_KEY=xxxxxxxxxxx
   ```

8. Now open up an ipython terminal and `import pyspark as ps`. If it did not throw an error,
   then you are set.
   

