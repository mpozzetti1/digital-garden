#nagios #terraform

# Which OS for the new server?

After reconsidering several times the new server's OS, we simply decided to follow up with an updated version of CentOS, instead of trying out RHEL or Ubuntu server.

### AWS Image Availability

As you can see, AWS doesn't natively support **CentOS** anymore.

In this case the solution is pretty simple, use the AMIs marketplace provided by AWS itself.

### CentOS 9 Stream

# Old Server Backup

Let's begin with the basic safety steps when doing this type of procedures, specially when we are working in a **production** server. Nagios provides different tools as the `backup_xi.sh` script which you can directly execute via the NagiosXI web interface.

We suggest to do the following:

- Manual AWS snapshot of the instance's volume.
- Instance AMI. (optional)
- Check last restore point in VEEAM for AWS or any backup tool you use.
- Save the Nagios backup file in a secure place.

### AWS Snapshot

1. Go to your instance and select the storage section.
2. Click the volume where your Nagios is installed.
3. Select it and then click in actions.
4. Click create snapshot.
5. Give it a good description to identify it later.

### Instance AMI (optional)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/515eee0b-17ce-454e-b866-54379c9a5c52/0ddd10c1-b0cc-452c-b9de-596f26edf1f2/image.png)

- After doing click on _Create Image, give it a good name and description._
- Click Save.

### VEEAM Restore Point

- Go to _Protected Data and search your instance._

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/515eee0b-17ce-454e-b866-54379c9a5c52/0e3835dd-48b0-43ee-bf0f-3fc1ce4a1503/image.png)

- Do click on the blue number to see all your restore points.
- Make sure everything is ok based on your backup requisites.

### Nagios Backup File

- Make sure you have **Administrator** access to the NagiosXI console.
- Go to the _**Admin**_ section.
- Scroll down until you find Local Backup Archives
- Before doing anything you need to perform a script in the Linux machine of your Nagios Core.

```bash
/usr/local/nagiosxi/scripts/repair_databases.sh
```

Wait until the process finishes and and return to the web console and create a backup.

If you want more control you can do it with this command:

```bash
/usr/local/nagiosxi/scripts/backup_xi.sh
```

The backups are always saved in `/store/backups/nagiosxi`.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/515eee0b-17ce-454e-b866-54379c9a5c52/08eea9a2-852d-4bb7-9d49-99f777393d95/image.png)

After the process is finished just download the file by clicking on the save icon, you wont be able to refresh the page until the download ends.

# New Instance using Terraform

First step, go to your bastion server that needs to be in AWS. We will need to configure the credentials to let this server perform **EC2** actions with no problems.

### AWS User and Policies

- Go to **IAM** and create a new user called `terraform`.
- Assign Full **EC2** and **VPC** access policies to it.
- Then go to the user configuration and create new credentials.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/515eee0b-17ce-454e-b866-54379c9a5c52/33b97368-7e22-44bb-802d-e6830505196b/image.png)

- Click on Create Access Key and select third party usage.
- Save the two values in a .txt file.
- Set the environmental variables with `root`.

```bash
export AWS_ACCESS_KEY_ID="ABCD123"
export AWS_SECRET_ACCESS_KEY="aBc123"
```

### Terraform File

We used this template for creating the new server, remember to subscribe to the Marketplace AMI before copying the AMI-Id and running the terraform.

```hcl
provider "aws" {
  region = "eu-west-1"
}

resource "aws_instance" "EC2Instance" {
  ami           = "ami-009f51225716cb42f"
  instance_type = "c6a.large"
  key_name      = "Infra_System_Production"
  monitoring    = false

  vpc_security_group_ids = ["sg-0e0f4499672c3ac7e"]
  subnet_id              = "subnet-a49a17fc"

  root_block_device {
    volume_size = 30
    volume_type = "gp3"
  }

  user_data = <<-EOF
  #!/bin/bash
  yum update -y
  yum install -y httpd
  systemctl start httpd
  systemctl enable httpd
  EOF

  tags = {
    Criticity    = "Moderate"
    Environment  = "Production"
    map-migrated = "d-server-03r8ncxvn4q9vx"
    Backup       = "AWS_Bck_EC2_Prod_NO_VSS_6PM"
    Application  = "Nagios XI Monitoring System"
    Comments     = "Monitoring System for Siram"
    Contact      = "it.siram.cloudteam.all.groups@veolia.com"
    OS           = "x"
    Role         = "Application Server"
    Service      = "INFRA"
    Name         = "nagiosxinew"
  }
}
```

### Run Terraform

- Move this `.tf` to your desired directory with root.
- Do the following commands one by one.

```bash
# Install terraform
yum install terraform

# Start terraform directory
terraform init

# See what the file will do
terraform plan

# Apply changes to run file
terraform apply
```

Remember that you can only have 1 terraform file per directory initiated.

# Nagios Fresh Installation

This is the easiest part, just follow this steps and you will have your new Nagios almost ready.

This will install all the dependencies needed for Nagios.

- Perform the following commands:

```bash
cd /tmp
wget <https://assets.nagios.com/downloads/nagiosxi/xi-latest.tar.gz>
tar xzf xi-latest.tar.gz
cd nagiosxi
./fullinstall
```

You will be prompted for the respective passwords for things like the database, I suggest you to use the same password for all the prompts for now, you can change it later.

# Restoring Backup File

Now the most important part, doing the restore of our previous configuration.

- First step is obviously moving the previously saved backup file into the `/tmp` folder.
- Then perform the following commands to start the restore, take into account to change the backup file name to yours.

```bash
/usr/local/nagiosxi/scripts/restore_xi.sh /tmp/backup-config-nagiosxi.tar.gz
```

Don't worry if you get this message before doing the restore:

```
WARNING: you are trying to restore a el7 backup on a el9 system
         Compiled plugins and other binaries as well as httpd configurations
         will NOT be restored.

         You will need to re-download the Nagios XI tarball, and re-install
         the subcomponents for this system. More info here:
         <https://assets.nagios.com/downloads/nagiosxi/docs/Backing-Up-And-Restoring-Nagios-XI.pdf>
```

### Restore Repair

Because we are changing OS, we need to perform an extra step to fix some binaries issues. In our case, we had some problems with the plugins, some of them didn't work, although they were in the`/usr/local/nagios/libexex.`

- Do the folllowing:

```bash
cd /tmp/
wget <https://assets.nagios.com/downloads/nagiosxi/scripts/restore_repair.sh>
chmod +x restore_repair.sh
./restore_repair.sh
```

### Database Repair

Final step is doing again the database repair. Just perform the following and NagiosXI will be restored succesfully:

```bash
/usr/local/nagiosxi/scripts/repair_databases.sh
systemctl restart nagios
```

# VMware Hosts Problem

Ww noticed we had some problems with our **VMware** hosts, they were marked as unknown. We had the following error:

```
VMWARE_API UNKNOWN - Failed to load module VMware::VIRuntime. Download and install 'VMware vSphere SDK for Perl', available at <https://my.vmware.com/group/vmware/downloads>
```

We tried to reach that website and similar ones but they were not available anymore, so we simply created an account into the Broadcom website and waited 2 hours to get access to download the files, sometimes if you don't wait after created a new account you cannot download files.

- From here you can get the SDK, choose the 7.0 version for x86, x64 Linux servers.
- [https://developer.broadcom.com/sdks/vsphere-perl-sdk/7.0?scrollString=vmware perl sdk](https://developer.broadcom.com/sdks/vsphere-perl-sdk/7.0?scrollString=vmware%20perl%20sdk)

### Prerequisites

```bash
yum install -y libxml2-devel libxml2 libuuid-devel perl-XML-LibXML perl-Env
export PERL_MM_USE_DEFAULT=1
cpan -i App::cpanminus
cpanm Crypt::SSLeay --dev
cpan -i Nagios::Monitoring::Plugin Nagios::Monitoring::Plugin::Functions
```

- Now, do the final commands to perform the final installation for the SDK.

```bash
cd /tmp
tar xzf VMware-vSphere*SDK*.tar.gz
cd vmware-vsphere-cli-distrib/
./vmware-install.pl EULA_AGREED=yes
```

- If you get a lack of dependencies you can use _**cpan.**_

```bash
sudo cpan
```

```bash
install Bundle::CPAN
```

- Now you should see the _**VMware Configuration Wizard**_ available.

# AWS Old Network Interface

Now that we have our NagiosXI well configured, we could terminate the old server and reuse the previous network interface to not build another network infrastructure again for the new server.

### New Server AMI

- After doing click on _Create Image, give it a good name and description._
- Click Save.

### Old server backup

- Skip this as we did it in the first steps.

### Terminate Old Server

- To free up the private IP from the old server, you need to terminate the instance in AWS.
- Go to Instance State and click _**Terminate**_.

### Launch New Server with AMI

- Go to your new server AMI, and launch an instance from it.
- Do all the same as you did before but change the private IP.
- Below Security Groups click on _**Advanced Network Configuration**_ and choose your old server private IP in the Primary IP section.

# Remove Previous Servers and Configs

- AWS servers.
- Networking Configurations and Interfaces.
- Remove Instance from Backup Tools.
- etc.

### Useful Files in the Troubleshooting

- /usr/local/nagios/var/nagios.log
- /usr/local/nagiosxi/scripts/repair_databases.sh
- /usr/local/nagiosxi/var/xi-sys.cfg
- /usr/local/nagios/etc/nagios.cfg
- /usr/local/nagios/libexec (plugins folder)