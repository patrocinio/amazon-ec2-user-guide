# Set up EC2 Instance Connect<a name="ec2-instance-connect-set-up"></a>

To use EC2 Instance Connect to connect to an instance, you need to configure every instance that will support a connection using Instance Connect \(this is a one\-time requirement for each instance\), and you need to grant permission to every IAM principal that will use Instance Connect\. After completing the following setup tasks, you can [connect to your instance using EC2 Instance Connect](ec2-instance-connect-methods.md)\.

**Topics**
+ [Task 1: Configure network access to an instance](#ec2-instance-connect-setup-security-group)
+ [Task 2: \(Conditional\) Install EC2 Instance Connect on an instance](#ec2-instance-connect-install)
+ [Task 3: \(Optional\) Install the EC2 Instance Connect CLI on your computer](#ec2-instance-connect-install-eic-CLI)
+ [Task 4: Configure IAM permissions for EC2 Instance Connect](#ec2-instance-connect-configure-IAM-role)

For more information about setting up EC2 Instance Connect, see [Securing your bastion hosts with Amazon EC2 Instance Connect](http://aws.amazon.com/blogs/infrastructure-and-automation/securing-your-bastion-hosts-with-amazon-ec2-instance-connect/)\.

**Limitations**
+ You can install EC2 Instance Connect on the following supported Linux distributions:
  + Amazon Linux 2 \(any version\)
  + Ubuntu 16\.04 or later
+ If you configured the `AuthorizedKeysCommand` and `AuthorizedKeysCommandUser` settings for SSH authentication, the EC2 Instance Connect installation will not update them\. As a result, you cannot use Instance Connect\.

**Prerequisites for installing EC2 Instance Connect**
+ **Verify the general prerequisites for connecting to your instance using SSH\.**

  For more information, see [General prerequisites for connecting to your instance](connection-prereqs.md)\.
+ **Install an SSH client on your local computer\.**

  Your local computer most likely has an SSH client installed by default\. You can check for an SSH client by typing ssh at the command line\. If your local computer doesn't recognize the command, you can install an SSH client\. For information about installing an SSH client on Linux or macOS X, see [http://www\.openssh\.com](http://www.openssh.com/)\. For information about installing an SSH client on Windows 10, see [OpenSSH in Windows](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_overview)\.
+ **Install the AWS CLI on your local computer\.**

  To configure the IAM permissions, you must use the AWS CLI\. For more information about installing the AWS CLI, see [Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html) in the *AWS Command Line Interface User Guide*\.
+ **\(Ubuntu\) Install the AWS CLI on your instance\.**

  To install EC2 Instance Connect on an Ubuntu instance, you must use the AWS CLI on the instance\. For more information about installing the AWS CLI, see [Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html) in the *AWS Command Line Interface User Guide*\.

## Task 1: Configure network access to an instance<a name="ec2-instance-connect-setup-security-group"></a>

You must configure the following network access so that your users can connect to your instance using EC2 Instance Connect:
+ If your users will access your instance over the internet, then your instance must have a public IP address and be in a public subnet\. For more information, see [Enable internet access](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#vpc-igw-internet-access) in the *Amazon VPC User Guide*\.
+ If your users will access your instance through the instance's private IP address, then you must establish private network connectivity to your VPC, such as by using AWS Direct Connect, AWS Site\-to\-Site VPN, or VPC peering, so that your users can reach the instance's private IP address\.
+ Ensure that the security group associated with your instance [allows inbound SSH traffic](authorizing-access-to-an-instance.md#add-rule-authorize-access) on port 22 from your IP address or from your network\. The default security group for the VPC does not allow incoming SSH traffic by default\. The security group created by the launch wizard allows incoming SSH traffic by default\. For more information, see [Authorize inbound traffic for your Linux instances](authorizing-access-to-an-instance.md)\.
+ \(Amazon EC2 console\) Ensure that the security group associated with your instance allows inbound SSH traffic from the IP address range for this service\. To identify the address range, download the JSON file provided by AWS and filter for the subset for EC2 Instance Connect, using `EC2_INSTANCE_CONNECT` as the service value\. For more information about downloading the JSON file and filtering by service, see [AWS IP address ranges](https://docs.aws.amazon.com/general/latest/gr/aws-ip-ranges.html) in the *Amazon Web Services General Reference*\.

## Task 2: \(Conditional\) Install EC2 Instance Connect on an instance<a name="ec2-instance-connect-install"></a>

You can skip this task if you used one of the following AMIs to launch your instance because they come preinstalled with EC2 Instance Connect:
+ Amazon Linux 2 2\.0\.20190618 or later
+ Ubuntu 20\.04 or later

For earlier versions of these AMIs, you must install Instance Connect on every instance that will support connecting using Instance Connect\.

Installing Instance Connect configures the SSH daemon on the instance\. The procedure for installing Instance Connect is different for instances launched using Amazon Linux 2 and Ubuntu\.

------
#### [ Amazon Linux 2 ]

**To install EC2 Instance Connect on an instance launched with Amazon Linux 2**

1. Connect to your instance using SSH\.

   Use the SSH key pair that was assigned to your instance when you launched it and the default user name of the AMI that you used to launch your instance\. For Amazon Linux 2, the default user name is `ec2-user`\.

   For example, if your instance was launched using Amazon Linux 2, your instance's public DNS name is `ec2-a-b-c-d.us-west-2.compute.amazonaws.com`, and the key pair is `my_ec2_private_key.pem`, use the following command to SSH into your instance:

   ```
   $ ssh -i my_ec2_private_key.pem ec2-user@ec2-a-b-c-d.us-west-2.compute.amazonaws.com
   ```

   For more information about connecting to your instance, see [Connect to your Linux instance using SSH](AccessingInstancesLinux.md)\.

1. Install the EC2 Instance Connect package on your instance\.

   For Amazon Linux 2, use the yum install command\.

   ```
   [ec2-user ~]$ sudo yum install ec2-instance-connect
   ```

   You should see three new scripts in the `/opt/aws/bin/` folder:

   ```
   eic_curl_authorized_keys
   eic_parse_authorized_keys
   eic_run_authorized_keys
   ```

1. \(Optional\) Verify that Instance Connect was successfully installed on your instance\.

   Use the sudo less command to check that the `/etc/ssh/sshd_config` file was correctly updated as follows:

   ```
   [ec2-user ~]$ sudo less /etc/ssh/sshd_config
   ```

   Instance Connect was successfully installed if the `AuthorizedKeysCommand` and `AuthorizedKeysCommandUser` lines in the `/etc/ssh/sshd_config` file contain the following values:

   ```
   AuthorizedKeysCommand /opt/aws/bin/eic_run_authorized_keys %u %f
   AuthorizedKeysCommandUser ec2-instance-connect
   ```
   + `AuthorizedKeysCommand` sets the `eic_run_authorized_keys` script to look up the keys from the instance metadata
   + `AuthorizedKeysCommandUser` sets the system user as `ec2-instance-connect`
**Note**  
If you previously configured `AuthorizedKeysCommand` and `AuthorizedKeysCommandUser`, the Instance Connect installation will not change the values and you will not be able to use Instance Connect\.

------
#### [ Ubuntu ]

**To install EC2 Instance Connect on an instance launched with Ubuntu 16\.04 or later**

1. Connect to your instance using SSH\.

   Use the SSH key pair that was assigned to your instance when you launched it and use the default user name of the AMI that you used to launch your instance\. For an Ubuntu AMI, the user name is `ubuntu`\.

   If your instance was launched using Ubuntu, your instance's public DNS name is `ec2-a-b-c-d.us-west-2.compute.amazonaws.com`, and the key pair is `my_ec2_private_key.pem`, use the following command to SSH into your instance:

   ```
   $ ssh -i my_ec2_private_key.pem ubuntu@ec2-a-b-c-d.us-west-2.compute.amazonaws.com
   ```

   For more information about connecting to your instance, see [Connect to your Linux instance using SSH](AccessingInstancesLinux.md)\.

1. \(Optional\) Ensure your instance has the latest Ubuntu AMI\.

   For Ubuntu, use the following commands to update all the packages on your instance\.

   ```
   ubuntu:~$ sudo apt-get update
   ```

   ```
   ubuntu:~$ sudo apt-get upgrade
   ```

1. Install the Instance Connect package on your instance\.

   For Ubuntu, use the sudo apt\-get command\.

   ```
   ubuntu:~$ sudo apt-get install ec2-instance-connect
   ```

   You should see three new scripts in the `/usr/share/ec2-instance-connect/` folder:

   ```
   eic_curl_authorized_keys
   eic_parse_authorized_keys
   eic_run_authorized_keys
   ```

1. \(Optional\) Verify that Instance Connect was successfully installed on your instance\.

   Use the sudo less command to check that the `/lib/systemd/system/ssh.service.d/ec2-instance-connect.conf` was correctly updated as follows:

   ```
   ubuntu:~$ sudo less /lib/systemd/system/ssh.service.d/ec2-instance-connect.conf
   ```

   Instance Connect was successfully installed if the `AuthorizedKeysCommand` and `AuthorizedKeysCommandUser` lines in the `/lib/systemd/system/ssh.service.d/ec2-instance-connect.conf` file contain the following values:

   ```
   AuthorizedKeysCommand /usr/share/ec2-instance-connect/eic_run_authorized_keys %%u %%f
   AuthorizedKeysCommandUser ec2-instance-connect
   ```
   + `AuthorizedKeysCommand` sets the `eic_run_authorized_keys` script to look up the keys from the instance metadata
   + `AuthorizedKeysCommandUser` sets the system user as `ec2-instance-connect`
**Note**  
If you previously configured `AuthorizedKeysCommand` and `AuthorizedKeysCommandUser`, the Instance Connect installation will not change the values and you will not be able to use Instance Connect\.

------

For more information about the EC2 Instance Connect package, see [aws/aws\-ec2\-instance\-connect\-config ](https://github.com/aws/aws-ec2-instance-connect-config) on the GitHub website\.

## Task 3: \(Optional\) Install the EC2 Instance Connect CLI on your computer<a name="ec2-instance-connect-install-eic-CLI"></a>

The EC2 Instance Connect CLI provides a simplified experience to connect to EC2 instances through a single command, `mssh instance_id`\. For more information, see [Connect using the EC2 Instance Connect CLI](ec2-instance-connect-methods.md#ec2-instance-connect-connecting-ec2-cli)\.

**Note**  
There is no need to install the EC2 Instance Connect CLI if users will only use the Amazon EC2 console or an SSH client to connect to an instance\.

**To install the EC2 Instance Connect CLI package**  
Use `[pip](https://docs.aws.amazon.com/cli/latest/userguide/install-linux.html#install-linux-pip)` to install the `ec2instanceconnectcli` package\. For more information, see [aws/aws\-ec2\-instance\-connect\-cli ](https://github.com/aws/aws-ec2-instance-connect-cli) on the GitHub website, and [https://pypi\.org/project/ec2instanceconnectcli/](https://pypi.org/project/ec2instanceconnectcli/) on the Python Package Index \(PyPI\) website\.

```
$ pip install ec2instanceconnectcli
```

## Task 4: Configure IAM permissions for EC2 Instance Connect<a name="ec2-instance-connect-configure-IAM-role"></a>

For your IAM principals to connect to an instance using EC2 Instance Connect, you must grant them permission to push the public key to the instance\. You grant them the permission by creating an IAM policy and attaching the policy to the IAM principals that require the permission\. For more information, see [Actions, resources, and condition keys for Amazon EC2 Instance Connect](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonec2instanceconnect.html)\.

The following instructions explain how to create the policy and attach it to an IAM user using the AWS CLI\. The same policy could be applied to other IAM principals, such as IAM roles\. For instructions that use the AWS Management Console, see [Creating IAM policies \(console\)](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html#access_policies_create-console.html), [Adding permissions by attaching policies directly to the user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_change-permissions.html#users_change_permissions-add-directly-console), and [Creating IAM roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html) in the *IAM User Guide*\.

**To grant an IAM principal permission for EC2 Instance Connect \(AWS CLI\)**

1. Create a JSON policy document that includes the following:
   + The `ec2-instance-connect:SendSSHPublicKey` action\. This grants an IAM principal permission to push the public key to an instance\. With `ec2-instance-connect:SendSSHPublicKey`, consider restricting access to specific EC2 instances\. Otherwise, all IAM principals with this permission can connect to all EC2 instances\. You can also restrict access by specifying resource ARNs or by using resource tags as [condition keys](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonec2instanceconnect.html#amazonec2instanceconnect-policy-keys)\.
   + The `ec2:osuser` condition\. This specifies the name of the OS user that can push the public key to an instance\. Use the default user name for the AMI that you used to launch the instance\. The default user name for Amazon Linux 2 is `ec2-user`, and for Ubuntu it's `ubuntu`\.
   + The `ec2:DescribeInstances` action\. This is required when using the EC2 Instance Connect CLI because the wrapper calls this action\. IAM principals might already have permission to call this action from another policy\.

   The following is an example policy document\. You can omit the statement for the `ec2:DescribeInstances` action if your users will only use an SSH client to connect to your instances\. You can replace the specified instances in `Resource` with the wildcard `*` to grant users access to all EC2 instances using EC2 Instance Connect\.

   ```
   {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Action": "ec2-instance-connect:SendSSHPublicKey",
           "Resource": [
               "arn:aws:ec2:region:account-id:instance/i-1234567890abcdef0",
               "arn:aws:ec2:region:account-id:instance/i-0598c7d356eba48d7"
           ],
           "Condition": {
               "StringEquals": {
                   "ec2:osuser": "ami-username"
               }
           }
         },
         {
           "Effect": "Allow",
           "Action": "ec2:DescribeInstances",
           "Resource": "*"
         }
       ]
   }
   ```

   The preceding policy allows access to specific instances, identified by their instance ID\. Alternatively, you can use resource tags to control access to an instance\. Attribute\-based access control is an authorization strategy that defines permissions based on tags that can be attached to users and AWS resources\. For example, the following policy allows an IAM user to access an instance only if that instance has a resource tag with key=`tag-key` and value=`tag-value`\. For more information about using tags to control access to your AWS resources, see [Controlling access to AWS resources](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_tags.html#access_tags_control-resources) in the *IAM User Guide*\.

   ```
   { 
      "Version":"2012-10-17",
      "Statement":[ 
         { 
            "Effect":"Allow",
            "Action":"ec2-instance-connect:SendSSHPublicKey",
            "Resource": "arn:aws:ec2:region:account-id:instance/*",
            "Condition":{ 
               "StringEquals":{ 
                  "aws:ResourceTag/tag-key":"tag-value"
               }
            }
         },
         {
           "Effect": "Allow",
           "Action": "ec2:DescribeInstances",
           "Resource": "*"
         }
      ]
   }
   ```

1. Use the [create\-policy](https://docs.aws.amazon.com/cli/latest/reference/iam/create-policy.html) command to create a new managed policy, and specify the JSON document that you created to use as the content for the new policy\.

   ```
   $ aws iam create-policy --policy-name my-policy --policy-document file://JSON-file-name
   ```

1. Use the [attach\-user\-policy](https://docs.aws.amazon.com/cli/latest/reference/iam/attach-user-policy.html) command to attach the managed policy to the specified IAM user\. For the `--user-name` parameter, specify the friendly name \(not the ARN\) of the IAM user\.

   ```
   $ aws iam attach-user-policy --policy-arn arn:aws:iam::account-id:policy/my-policy --user-name IAM-friendly-name
   ```