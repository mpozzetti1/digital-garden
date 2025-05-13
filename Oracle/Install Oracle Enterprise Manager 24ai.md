#oracle #oem24ai

This document describes the technical details of the installation and configuration of **Oracle Enterprise Manager 24 AI (OEM24AI)** on a new server, **SRVSVOEM2**, running **Oracle Linux 8 (OL8)** in the **Oracle Cloud Infrastructure (OCI)**. The server is located in the **Management** compartment.

Key components of this setup include:

- **Oracle Database 19.24 Enterprise Edition** (Multitenant Architecture)
    - **Container Database (CDB):** `oemrepo4`
    - **Pluggable Database (PDB):** `oempdb` (Repository for OEM)
- **OEM24AI Software** located at:
    - **ORACLE_HOME** = `/home/oracle`
    - **OEM_HOME** = `/home/oracle/middleware/oms_home`
- Access to OEM24AI URL: `https://localhost:7803/em`
- Database SYS password: `Davide123`
- OEM SYSMAN user password: `Davide123`
- Some useful files located in: `/oracle/home`
- Firewalld service disabled to allow OEM operation
- Port **3872** must be free for OEM Agent remote setup

## 2. What Is an OEM Repository Database?

An **Oracle Enterprise Manager Repository Database** is a dedicated database (or a dedicated pluggable database within a container) that stores all the configuration data, monitoring data, and information related to the targets managed by Oracle Enterprise Manager. This repository holds critical information such as:

- Metrics and performance data gathered from monitored targets
- Configuration details, job scheduling, and historical data
- Users, roles, and security settings for OEM

In this environment, `oempdb` (a pluggable database inside the container `oemrepo4`) is the dedicated repository that OEM uses to store all necessary information to manage and monitor targets across your infrastructure.

## 3. Environment Details

1. **Server Name (OCI):** `SRVSVOEM2`
2. **Operating System:** Oracle Linux 8 (OL8)
3. **Compartment:** Management
4. **Oracle Database Version:** 19.24 Enterprise Edition (Containerized)
    - **CDB Name:** `oemrepo4`
    - **PDB Name:** `oempdb`
5. **OEM Version:** 24 AI
6. **Oracle User:** `oracle` (all installations and configurations performed under this user)
7. **ORACLE_HOME:** `/home/oracle`
8. **OEM_HOME (OMS Home):** `/home/oracle/middleware/oms_home`
9. **Repository Database SYS Password:** `Davide123`
10. **OEM Administrator (SYSMAN) Credentials:**
    - **User:** `sysman`
    - **Password:** `Davide123`
11. **Network Access/Firewall:**
    - `firewalld` disabled to allow inbound/outbound OEM connections
    - OEM HTTPS console port: `7803`
    - **OEM Agent Port** to be used for remote agent connectivity: `3872` (must be open / not in use by a previous agent)
12. **Useful Additional Files/Logs Location:** `/oracle/home`

## 4. Installation and Configuration Steps

### 4.1. Database Setup (oemrepo4 Container and oempdb Pluggable)

1. **Create Container Database (If not already created)**
    - Not detailed here if this was done prior, but typically you would use DBCA or command-line DB creation scripts.
2. **Create Pluggable Database `oempdb`**
    - Once the CDB is created, you create a PDB (oempdb) using standard SQL statements or DBCA.
3. **Open the PDB in Read-Write Mode** and ensure it’s set to open on database startup:
    
```sql
ALTER PLUGGABLE DATABASE oempdb OPEN;
ALTER PLUGGABLE DATABASE oempdb SAVE STATE;
```
    
4. **Verify TNS listener settings** so that the pluggable database can be accessed by OEM installation scripts. Typically, `listener.ora` and `tnsnames.ora` are located under your `$ORACLE_HOME/network/admin` directory.

### 4.2. OEM24ai Software Installation

1. **Prerequisites**
    - Ensure the system meets the resource requirements (CPU, memory, disk space) for OEM24AI.
    - Disable SELinux or configure as recommended by Oracle documentation.
    - **Disable firewalld** or open appropriate ports (for demonstration, we have disabled firewalld):
        
```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```
        
Confirm the environment variable settings under the `oracle` user:        
    
```bash
export ORACLE_HOME=/home/oracle
export OMS_HOME=/home/oracle/middleware/oms_home
export PATH=$ORACLE_HOME/bin:$PATH
export JAVA_HOME=/home/oracle/java (or the path where Java is installed, if separate)
```
    
        
2. **Install the OEM Binaries** under `/home/oracle/middleware/oms_home`
    - Unpack the OEM installer (for example, the OEM24AI zip) into a staging directory (`/home/oracle/stage`).
    - Run the installer:
        
```bash
cd /home/oracle/stage
./runInstaller
```
        
Follow the prompts, specifying the **OMS Home** as `/home/oracle/middleware/oms_home`.

3. **Repository Configuration**
    - During the configuration wizard, specify the connection details to the **oempdb** PDB:
        - **Hostname/Service**: The host or IP of the container DB plus the service name for `oempdb`
        - **SYS password**: `Davide123`
        - **Port**: Typically 1521 (adjust if custom)
    - The installer will automatically create the OEM repository schema in `oempdb`.
    - Confirm the repository creation completes successfully.

### 4.3. Post-Installation Tasks

1. **Validate the Installation**
    - Upon successful installation, OEM should provide the URL to access the console, in this case:
        
```
https://localhost:7803/em
```
        
2. **Check the OMS and Agent Status**
    - After installation, ensure that the OMS is up and running:
        
```bash
cd /home/oracle/middleware/oms_home/bin
./emctl status oms
```

if the OEM application server is up and the ports are correctly listening.

## 5. Starting and Stopping OEM (OMS)

Below are common commands used for controlling the Oracle Management Service (OMS):

1. **Navigate to OMS Home bin directory**
    
```bash
cd /home/oracle/middleware/oms_home/bin
```
    
2. **Check Status**
    
```bash
./emctl status oms
```
    
3. **Start OMS**
    
```bash
./emctl start oms
```
    
4. **Stop OMS**
    
```bash
./emctl stop oms
```
    
5. **Restart OMS** (Stop then Start)

```bash
./emctl stop oms
./emctl start oms
```
    
## 6. Additional Notes and Best Practices

1. **Security Considerations**
    - Re-enable `firewalld` if needed, but ensure that required ports (e.g., 1521 for DB, 7803 for OMS HTTPS, 3872 for Agent) are allowed.
    - Consider SSL/TLS certificates management for your OEM console.
    - Change all default passwords (SYS, SYSMAN, etc.) to secure, strong passwords if used in production.
2. **Routine Maintenance**
    - Regularly monitor disk usage in both the OMS Home and the repository database.
    - Apply quarterly PSU/BP patches for Oracle DB 19c and relevant OEM patches.
3. **Performance Tuning**
    - Depending on the number of targets you are monitoring, ensure the repository database `oempdb` has adequate memory, CPU, and storage resources allocated.
    - For large environments, separate the repository database onto dedicated storage or a dedicated server if necessary.

## 7. Target Database Setup

### IaaS Database Instance

Before proceeding, ensure that the firewall configurations between the **host** and the **target** servers are properly set up. Refer to the diagram below for all required ports.

Next, on the **target server**, add the following record to `/etc/hosts` so it can resolve the **host** server name during agent installation:

```
localhost srvsvoem2.mansubnet.manvcn.oraclevcn.com srvsvoem2
```

### Steps to Add the Target Manually

1. In the Oracle Enterprise Manager (OEM) console, navigate:
    
    > Setup
    > 
    > 
    > > Add Target
    > > 
    > > 
    > > > Add Target Manually
    > > > 
2. Click **Add** to initiate the process.
3. Enter the **IP** of the target server.
4. Choose the appropriate **Operating System**.
5. For **Installation Base Directory**, specify the path where you want the agent to be installed.
6. The **Instance Directory** is automatically set.
7. Under **Named Credentials**, add an **SSH key credential**. Provide the **username** and the **private key** file. Then, set **Run Privilege** to `sudo` and select the user who will own the agent process (for example, `oracle` or `oradba`).
8. Leave other settings as default, then click **Next**.
9. After the process finishes, click **Done**. Return to:
    
    > Add Target Manually
    > 
    > 
    > > Add Non-Host Targets Manually
    > > 
10. Search for the **host** you just added, select it, and choose **Database Instance**.

### Unlock and Configure the `dbsnmp` User

On the **database server**, unlock the `dbsnmp` user and set a memorable password. For example:

```sql
ALTER USER dbsnmp ACCOUNT UNLOCK;
ALTER USER dbsnmp IDENTIFIED BY YourNewPassword;
```

> Note: Replace YourNewPassword with a secure password of your choice.
> 

### Final Steps

1. Fill in the required information for the **database instance**.
    - Make sure the **Role** field is set to **Normal**.
2. Click **Test Connection**. If the connection is successful, proceed.
3. Click **Next** to complete adding the **database instance** to OEM.

### Container Database Instance

The **host configuration** (firewall setup and editing `/etc/hosts`) remains **the same** as in the regular (non-container) database instance setup. Refer to the previous steps for details on how to:

- Configure the firewall to allow required ports.
- Update the `/etc/hosts` file with the correct host entry.

Once those steps are complete, proceed as follows:

1. In the Oracle Enterprise Manager (OEM) console, navigate:
    
    > Targets
    > 
    > 
    > > Databases
    > > 
    > > 
    > > > Add Database
    > > > 
2. Select your **target host** from the list and wait for the target details to load.
3. Choose the **Pluggable Databases** and the **Listeners** you want to monitor.
4. Click **Next**.
5. Now your Pluggable Databases are available to monitor.

### Amazon Oracle RDS Instance

### 1. Configure the Option Group

1. **Create or modify an Option Group** in your Amazon RDS console to include the **OEM** option (not the **OEM Agent** option).
2. Navigate to your RDS instance, choose **Modify**, and assign the newly created (or updated) Option Group.
3. If you select **Apply Immediately**, the instance will be **restarted** to apply changes.
4. This setup allows use of the **dbsnmp** user on non-container RDS instances.
    
    > Note: If your RDS instance is a Container Database (CDB), you cannot unlock the dbsnmp user since it resides in the CDB and RDS does not allow direct access to the container’s root.
    > 

### 2. Use a Bridge Server for OEM Integration

Because RDS instances cannot be added the same way as regular IaaS or DBCS servers, you need to use a **bridge server** as your OEM host:

1. **Identify an existing OEM-monitored host** (where you have already installed an Agent). Typically, this is a server hosting another database or another OEM-monitored component.
2. In the OEM console, go to:
    
    > Setup
    > 
    > 
    > > Add Target
    > > 
    > > 
    > > > Add Non-Host Targets Manually
    > > > 
3. Select the **bridge host** from the list.
4. Choose **Database Instance** as the target type.
5. Complete all the details using the **RDS connection string** and relevant credentials.
6. Click **Next**, and once the connection is verified, your **Amazon RDS instance** will appear in the **OEM** console.

## Repository Database Setup

### 1. Download the Database Package

- **Visit Oracle Software Delivery:** Go to [Oracle Software Delivery](https://edelivery.oracle.com/) and search for the Oracle Database version you need.
- **Download `wget.sh`:**Once you find your database, download the provided `wget.sh` file. This script will help you fetch the necessary packages on your Linux system.

### 2. Prepare the `wget.sh` File

- **Make It Executable:** After transferring the file to your Linux machine, run: This command ensures that the script is executable.
    
```bash
chmod +x wget.sh
```
    
### 3. Install Display Export Packages

- **Install Xorg (and related packages):** To support display forwarding, install the required packages. For example, on Oracle Linux or similar distributions: Ensure these packages are installed so that graphical output can be forwarded correctly.
    
```bash
sudo yum install xorg-x11-apps
```
    
### 4. Configure MobaXterm for Display Forwarding

- **Enable Display Forwarding:** Open MobaXterm and run: This command allows remote X11 clients to connect to your display.
    
```bash
xhost +
```

### 5. Set the DISPLAY Environment Variable

- **Export the Display Variable:** MobaXterm will provide you with an IP address for the display. Use that to export your `DISPLAY` variable. For example: Replace `192.168.5.216` with the IP address given by MobaXterm if different.
    
```bash
export DISPLAY='192.168.5.216:0.0'
```
    
### 6. Emulate Oracle Linux 7 Environment

- **Set the CV_ASSUME_DISTID Variable:** To ensure compatibility with Oracle's installer, assume your system is Oracle Linux 7:
    
```bash
export CV_ASSUME_DISTID=OEL7.8
```
    
### 7. Run the Installer

- **Start the Installation Process:** Finally, execute the installer by running:
    
```bash
./runInstaller
```

## OEM Email Notifications

### 1. Configure Mail Server Settings

1. **Log In:**
    Sign in to OEM24ai as a Super Administrator.
    
2. **Navigate to Mail Server Setup:**
    Go to **Setup > Notifications > Mail Servers**.
    
3. **Set Up Sender Identity:**
    - In the **Sender Identity** section, click **Add** (or **Edit**) and enter:
        - **Display Name** (e.g., "OEM24ai Notifications")
        - **Sender's Email Address** (e.g., admin@example.com)
4. **Configure Outgoing Mail (SMTP) Servers:**
    - In the **Outgoing Mail (SMTP) Servers** section, click **Add**.
    - Enter the SMTP server details:
        - **SMTP Server Hostname & Port** (e.g., smtp.example.com:587)
        - **Authentication Credentials** (if required)
        - **Encryption Method** (None, SSL, or TLS)
    - Click **Test Mail Servers** to verify that a test email is sent to the Sender's Email Address.
    - Click **Apply** to save your settings.
    - In this case we used Amazon SES as the SFTP server.

### 2. Configure Administrator Email Addresses

1. **Access Administrator Email Settings:**
    
    Click your username in the top-right corner and select **Enterprise Manager Password & E-mail**.
    
2. **Add Email Addresses:**
    - On the **Enterprise Manager Password & E-mail** page, scroll to the **E-Mail Addresses** section.
    - Click **Add Another Row** and enter the email addresses of all administrators who should receive notifications.
    - Click **Apply** to save the changes.

### 3. Create an Incident Notification Rule

1. **Navigate to Incident Rules:**
    
    Go to **Setup > Incidents > Incident Rules**.
    
2. **Create or Edit a Rule:**
    - Click **Create Rule** (or select an existing rule to edit).
    - Define a rule that monitors the status of your targets.
3. **Set Rule Criteria:**
    - **Target Selection:** Choose **Database Systems** and **Database Instances**.
4. **Configure Notification Action:**
    - On the **Actions** page, check the option to **Send Me E-Mail**.
    - Select SYSMAN to receive the emails.
    - (Optional) Add other notification methods if needed.
5. **Finalize the Rule:**
    
    Click **Finish** (or **Apply**) to activate the rule.