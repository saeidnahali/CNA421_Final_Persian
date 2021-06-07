# CNA421
## Final project
###  Saeid Nahali, Mohammad Pakizeh jam

## 1. Install and Configure Wordpress in AWS EC2 Ubuntu Server Instance
## 2. Create RDS instance and Connect from EC2
## 3. Get lambda instance talking to web server and RDS

**********************************************************

 ## Install and Configure Wordpress in AWS EC2 Ubuntu Server Instance

 Create an EC2 instance and install a web server
 In this step, you create a web server to connect to the Amazon RDS DB instance that you created in Create a DB instance.

 Launch an EC2 instance
 First, you create an Amazon EC2 instance in the public subnet of your VPC.

 To launch an EC2 instance

 Sign in to the AWS Management Console and open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.

 Choose EC2 Dashboard, and then choose Launch instance, as shown following.


                                EC2 Dashboard
                            
 Choose the Amazon Linux 2 AMI.


                            Choose an Amazon Machine Image
                        
 Choose the t2.micro instance type, as shown following, and then choose Next: Configure Instance Details.


                                Choose an Instance Type
                            
 On the Configure Instance Details page, shown following, set these values and keep the other values as their defaults:

 Network: Choose the VPC with both public and private subnets that you chose for the DB instance, such as the vpc-identifier | tutorial-vpc created in Create a VPC with private   and public subnets.

 Subnet: Choose an existing public subnet, such as subnet-identifier | Tutorial public | us-west-2a created in Create a VPC security group for a public web server.

 Auto-assign Public IP: Choose Enable.


                                Configure Instance Details
                            
 Choose Next: Add Storage.

 On the Add Storage page, keep the default values and choose Next: Add Tags.

 On the Add Tags page, shown following, choose Add Tag, then enter Name for Key and enter tutorial-web-server for Value.


                                Tag Instance
                            
 Choose Next: Configure Security Group.

 On the Configure Security Group page, shown following, choose Select an existing security group. Then choose an existing security group, such as the tutorial-securitygroup created in Create a VPC security group for a public web server. Make sure that the security group that you choose includes inbound rules for Secure Shell (SSH) and HTTP      access.


                                Configure Security Group
                            
 Choose Review and Launch.

 On the Review Instance Launch page, shown following, verify your settings and then choose Launch.


                                Review Instance Launch
                            
 On the Select an existing key pair or create a new key pair page, shown following, choose Create a new key pair and set Key pair name to tutorial-key-pair. Choose Download Key   Pair, and then save the key pair file on your local machine. You use this key pair file to connect to your EC2 instance.


                                Select an Existing Key Pair or Create a New Key Pair
                            
 To launch your EC2 instance, choose Launch Instances. On the Launch Status page, shown following, note the identifier for your new EC2 instance, for example: i-  0288d65fd4470b6a9.


                                Launch Status
                            
 Choose View Instances to find your instance.

 Wait until Instance Status for your instance reads as Running before continuing.

 Install an Apache web server with PHP
 Next, you connect to your EC2 instance and install the web server.

 To connect to your EC2 instance and install the Apache web server with PHP

 Connect to the EC2 instance that you created earlier by following the steps in Connect to your Linux instance.

 Get the latest bug fixes and security updates by updating the software on your EC2 instance. To do this, use the following command.

 Note
 The -y option installs the updates without asking for confirmation. To examine updates before installing, omit this option.

 sudo yum update -y
 After the updates complete, install the PHP software using the amazon-linux-extras install command. This command installs multiple software packages and related dependencies at the same time.

 sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
 If you receive an error stating sudo: amazon-linux-extras: command not found, then your instance was not launched with an Amazon Linux 2 AMI (perhaps you are using the Amazon  Linux AMI instead). You can view your version of Amazon Linux using the following command.

 cat /etc/system-release
 For more information, see Updating instance software.

 Install the Apache web server.

 sudo yum install -y httpd
 Start the web server with the command shown following.

 sudo systemctl start httpd
 You can test that your web server is properly installed and started. To do this, enter the public Domain Name System (DNS) name of your EC2 instance in the address bar of a web browser, for example: http://ec2-42-8-168-21.us-west-1.compute.amazonaws.com. If your web server is running, then you see the Apache test page.

 If you don't see the Apache test page, check your inbound rules for the VPC security group that you created in Tutorial: Create an Amazon VPC for use with a DB instance. Make sure that your inbound rules include a rule allowing HTTP (port 80) access for the IP address you use to connect to the web server.

 Note
 The Apache test page appears only when there is no content in the document root directory, /var/www/html. After you add content to the document root directory, your content appears at the public DNS address of your EC2 instance instead of the Apache test page.

 Configure the web server to start with each system boot using the systemctl command.

 sudo systemctl enable httpd
 To allow ec2-user to manage files in the default root directory for your Apache web server, modify the ownership and permissions of the /var/www directory. There are many ways to accomplish this task. In this tutorial, you add ec2-user to the apache group, to give the apache group ownership of the /var/www directory and assign write permissions to the group.

 To set file permissions for the Apache web server

 Add the ec2-user user to the apache group.

 sudo usermod -a -G apache ec2-user
 Log out to refresh your permissions and include the new apache group.

 exit
 Log back in again and verify that the apache group exists with the groups command.

 groups
 Your output looks similar to the following:

 ec2-user adm wheel apache systemd-journal
 Change the group ownership of the /var/www directory and its contents to the apache group.

 sudo chown -R ec2-user:apache /var/www
 Change the directory permissions of /var/www and its subdirectories to add group write permissions and set the group ID on subdirectories created in the future.

 sudo chmod 2775 /var/www
 find /var/www -type d -exec sudo chmod 2775 {} \;
 Recursively change the permissions for files in the /var/www directory and its subdirectories to add group write permissions.

 find /var/www -type f -exec sudo chmod 0664 {} \;
 Now, ec2-user (and any future members of the apache group) can add, delete, and edit files in the Apache document root, enabling you to add content, such as a static website or a PHP application.

 Note
 A web server running the HTTP protocol provides no transport security for the data that it sends or receives. When you connect to an HTTP server using a web browser, the URLs that you visit, the content of web pages that you receive, and the contents (including passwords) of any HTML forms that you submit are all visible to eavesdroppers anywhere along the network pathway. The best practice for securing your web server is to install support for HTTPS (HTTP Secure), which protects your data with SSL/TLS encryption. For more information, see Tutorial: Configure SSL/TLS with the Amazon Linux AMI in the Amazon EC2 User Guide.

 Connect your Apache web server to your DB instance
 Next, you add content to your Apache web server that connects to your Amazon RDS DB instance.

 To add content to the Apache web server that connects to your DB instance

 While still connected to your EC2 instance, change the directory to /var/www and create a new subdirectory named inc.
```
 cd /var/www
 mkdir inc
 cd inc
 ```
 Create a new file in the inc directory named dbinfo.inc, and then edit the file by calling nano (or the editor of your choice).
```
>dbinfo.inc
nano dbinfo.inc
```
 Add the following contents to the dbinfo.inc file. Here, db_instance_endpoint is your DB instance endpoint, without the port, and master password is the master password for your DB instance.

 Note
 We recommend placing the user name and password information in a folder that isn't part of the document root for your web server. Doing this reduces the possibility of your security information being exposed.
```
<?php

define('DB_SERVER', 'db_instance_endpoint');
define('DB_USERNAME', 'tutorial_user');
define('DB_PASSWORD', 'master password');
define('DB_DATABASE', 'sample');

?>
```                
Save and close the dbinfo.inc file.

Change the directory to /var/www/html.
```
cd /var/www/html
```
Create a new file in the html directory named SamplePage.php, and then edit the file by calling nano (or the editor of your choice).
```
>SamplePage.php
nano SamplePage.php
```                
 Add the following contents to the SamplePage.php file:

 Note
 We recommend placing the user name and password information in a folder that isn't part of the document root for your web server. Doing this reduces the possibility of your security information being exposed.

```
<?php include "../inc/dbinfo.inc"; ?>
<html>
<body>
<h1>Sample page</h1>
<?php

  /* Connect to MySQL and select the database. */
  $connection = mysqli_connect(DB_SERVER, DB_USERNAME, DB_PASSWORD);

  if (mysqli_connect_errno()) echo "Failed to connect to MySQL: " . mysqli_connect_error();

  $database = mysqli_select_db($connection, DB_DATABASE);

  /* Ensure that the EMPLOYEES table exists. */
  VerifyEmployeesTable($connection, DB_DATABASE);

  /* If input fields are populated, add a row to the EMPLOYEES table. */
  $employee_name = htmlentities($_POST['NAME']);
  $employee_address = htmlentities($_POST['ADDRESS']);

  if (strlen($employee_name) || strlen($employee_address)) {
    AddEmployee($connection, $employee_name, $employee_address);
  }
?>

<!-- Input form -->
<form action="<?PHP echo $_SERVER['SCRIPT_NAME'] ?>" method="POST">
  <table border="0">
    <tr>
      <td>NAME</td>
      <td>ADDRESS</td>
    </tr>
    <tr>
      <td>
        <input type="text" name="NAME" maxlength="45" size="30" />
      </td>
      <td>
        <input type="text" name="ADDRESS" maxlength="90" size="60" />
      </td>
      <td>
        <input type="submit" value="Add Data" />
      </td>
    </tr>
  </table>
</form>

<!-- Display table data. -->
<table border="1" cellpadding="2" cellspacing="2">
  <tr>
    <td>ID</td>
    <td>NAME</td>
    <td>ADDRESS</td>
  </tr>

<?php

$result = mysqli_query($connection, "SELECT * FROM EMPLOYEES");

while($query_data = mysqli_fetch_row($result)) {
  echo "<tr>";
  echo "<td>",$query_data[0], "</td>",
       "<td>",$query_data[1], "</td>",
       "<td>",$query_data[2], "</td>";
  echo "</tr>";
}
?>

</table>

<!-- Clean up. -->
<?php

  mysqli_free_result($result);
  mysqli_close($connection);

?>

</body>
</html>


<?php

/* Add an employee to the table. */
function AddEmployee($connection, $name, $address) {
   $n = mysqli_real_escape_string($connection, $name);
   $a = mysqli_real_escape_string($connection, $address);

   $query = "INSERT INTO EMPLOYEES (NAME, ADDRESS) VALUES ('$n', '$a');";

   if(!mysqli_query($connection, $query)) echo("<p>Error adding employee data.</p>");
}

/* Check whether the table exists and, if not, create it. */
function VerifyEmployeesTable($connection, $dbName) {
  if(!TableExists("EMPLOYEES", $connection, $dbName))
  {
     $query = "CREATE TABLE EMPLOYEES (
         ID int(11) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
         NAME VARCHAR(45),
         ADDRESS VARCHAR(90)
       )";

     if(!mysqli_query($connection, $query)) echo("<p>Error creating table.</p>");
  }
}

/* Check for the existence of a table. */
function TableExists($tableName, $connection, $dbName) {
  $t = mysqli_real_escape_string($connection, $tableName);
  $d = mysqli_real_escape_string($connection, $dbName);

  $checktable = mysqli_query($connection,
      "SELECT TABLE_NAME FROM information_schema.TABLES WHERE TABLE_NAME = '$t' AND TABLE_SCHEMA = '$d'");

  if(mysqli_num_rows($checktable) > 0) return true;

  return false;
}
?>                        
```                
 Save and close the SamplePage.php file.

 Verify that your web server successfully connects to your DB instance by opening a web browser and browsing to http://EC2 instance endpoint/SamplePage.php, for example: http://ec2-55-122-41-31.us-west-2.compute.amazonaws.com/SamplePage.php.

 You can use SamplePage.php to add data to your DB instance. The data that you add is then displayed on the page. To verify that the data was inserted into the table, you can install MySQL on the Amazon EC2 instance, connect to the DB instance, and query the table.

 To make sure that your DB instance is as secure as possible, verify that sources outside of the VPC can't connect to your DB instance.

 After you have finished testing your web server and your database, you should delete your DB instance and your Amazon EC2 instance.

 To delete a DB instance, follow the instructions in Deleting a DB instance. You don't need to create a final snapshot.

 To terminate an Amazon EC2 instance, follow the instruction in Terminate your instance in the Amazon EC2 User Guide.
