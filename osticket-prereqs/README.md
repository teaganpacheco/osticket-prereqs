# osTicket: Prerequisites and Installation

This project walks through the deployment and installation of **osTicket**, an open-source help desk ticketing system, in a Microsoft Azure lab environment.

The goal is not only to install osTicket successfully, but also to understand the infrastructure and support technologies that make a web-based ticketing system work: **Windows Server, IIS, PHP, MySQL, networking, permissions, and application validation**.

> **CourseCareers IT Lab 1:** Deploy and Install osTicket

---

## Project Objectives

By completing this project, you will:

- Deploy a Windows Server virtual machine in Microsoft Azure
- Connect to the server using Remote Desktop Protocol (RDP)
- Install and configure Internet Information Services (IIS)
- Install PHP and register it with IIS
- Install and configure MySQL
- Create a dedicated osTicket database and database user
- Install osTicket
- Complete the osTicket web installer
- Apply basic post-installation security controls
- Validate and troubleshoot the completed deployment

---

## Technologies Used

- Microsoft Azure
- Azure Virtual Machines
- Azure Network Security Groups
- Windows Server 2022
- Remote Desktop Protocol (RDP)
- Internet Information Services (IIS)
- PHP 8.4
- MySQL 8.0
- osTicket 1.18.x
- PowerShell / Command Prompt
- MySQL Workbench, HeidiSQL, or MySQL CLI

---

## Lab Architecture

```text
Your Computer
     |
     | RDP / TCP 3389
     v
Microsoft Azure
     |
     v
Windows Server 2022 VM
     |
     +-- IIS
     |    |
     |    +-- PHP
     |    |
     |    +-- osTicket
     |
     +-- MySQL
          |
          +-- osticket database
          +-- osticket_user
```

For this training lab, all application components run on one virtual machine.

---

## Prerequisites

Before beginning, you should have:

- A Microsoft Azure account with permission to create resources
- A computer with Remote Desktop capability
- Internet access from the Azure VM
- Basic familiarity with Windows administration
- A password manager or another secure location for storing lab credentials
- Enough Azure quota to deploy a small Windows Server VM

### Recommended Azure VM Configuration

| Setting | Recommended Value |
|---|---|
| VM Name | `osticket-vm` |
| Operating System | Windows Server 2022 Datacenter |
| Architecture | x64 |
| vCPU | 2 or more |
| Memory | 8 GB recommended |
| Public IP | Enabled for the lab |
| RDP | TCP 3389 restricted to your public IP |
| Disk | Standard SSD is sufficient |

> **Security note:** Do not expose RDP to the entire internet if you can avoid it. Restrict TCP 3389 to your current public IP address through the VM's Network Security Group.

---

# Part 1 — Deploy the Azure Virtual Machine

## 1. Create the VM

In the Azure portal:

1. Navigate to **Virtual Machines**.
2. Select **Create > Azure virtual machine**.
3. Create or select a Resource Group.
4. Configure:
   - **Virtual machine name:** `osticket-vm`
   - **Image:** Windows Server 2022 Datacenter
   - **Size:** A small 2-vCPU or larger lab-appropriate size
5. Create a local administrator username.
6. Generate a strong, unique password and save it securely.
7. Allow **RDP (3389)** only as needed.
8. Review and create the VM.

## 2. Connect with Remote Desktop

After deployment:

1. Open the VM in the Azure portal.
2. Record its public IP address.
3. Launch **Remote Desktop Connection** on your computer.
4. Connect to the public IP address.
5. Authenticate using the Windows administrator account you created.

### Verify Your Work

You should be able to:

- Sign in to `osticket-vm`
- Open Server Manager
- Browse the internet from the VM

---

# Part 2 — Install Internet Information Services (IIS)

osTicket is a PHP web application. IIS will provide the web-server component of the environment.

## Install IIS

1. Open **Server Manager**.
2. Select **Manage > Add Roles and Features**.
3. Choose **Role-based or feature-based installation**.
4. Select the local server.
5. Enable **Web Server (IIS)**.
6. Under:

```text
Web Server
└── Application Development
    └── CGI
```

Enable **CGI**.

7. Complete the installation.

## Validate IIS

Open a browser on the VM and navigate to:

```text
http://localhost
```

You should see the default IIS landing page.

### Optional PowerShell Validation

```powershell
Get-Service W3SVC
```

The `W3SVC` service should be running.

---

# Part 3 — Install PHP and IIS Components

For the current osTicket 1.18 series, use a supported PHP 8.x release. This lab standardizes on **PHP 8.4 x64 Non-Thread-Safe**.

## Components

Install:

- Microsoft Visual C++ Redistributable required by your PHP build
- IIS URL Rewrite Module
- PHP Manager 2 for IIS, if using the GUI workflow
- PHP 8.4 x64 Non-Thread-Safe

## Create the PHP Directory

Create:

```text
C:\PHP
```

Extract PHP into that directory.

You should eventually have:

```text
C:\PHP\php.exe
C:\PHP\php-cgi.exe
C:\PHP\php.ini-production
```

## Create php.ini

Copy:

```text
C:\PHP\php.ini-production
```

to:

```text
C:\PHP\php.ini
```

## Register PHP with IIS

Using PHP Manager:

1. Open **IIS Manager**.
2. Select the server.
3. Open **PHP Manager**.
4. Register a new PHP version.
5. Select:

```text
C:\PHP\php-cgi.exe
```

## Enable Required PHP Extensions

osTicket will display prerequisite warnings if extensions are missing.

Commonly required or recommended extensions include:

```text
mysqli
ctype
fileinfo
gd
gettext
iconv
imap
intl
mbstring
opcache
phar
xml
zip
```

Edit `php.ini` or use PHP Manager to enable the extensions present in your PHP distribution.

## Restart IIS

Run from an elevated Command Prompt or PowerShell session:

```powershell
iisreset
```

## Validate PHP

Run:

```powershell
C:\PHP\php.exe -v
```

Then list loaded modules:

```powershell
C:\PHP\php.exe -m
```

### Verify Your Work

Confirm that:

- IIS is running
- `php.exe -v` returns the expected PHP version
- IIS recognizes `php-cgi.exe`
- Required PHP extensions are enabled

---

# Part 4 — Install MySQL

Install **MySQL Community Server 8.0**.

During setup:

1. Choose an installation profile appropriate for the lab.
2. Configure MySQL as a Windows service.
3. Configure the service to start automatically.
4. Set a strong password for the MySQL `root` account.
5. Store the password securely.

For this single-server lab, keep the database local to the osTicket VM.

## Validate MySQL

In PowerShell:

```powershell
Get-Service MySQL*
```

You should see the MySQL service in a running state.

---

# Part 5 — Create the osTicket Database

Do **not** configure osTicket to run as the MySQL `root` account.

Instead, create:

- Database: `osticket`
- Application account: `osticket_user`

You may use MySQL Workbench, HeidiSQL, or the MySQL CLI.

## Example SQL

Sign in as the MySQL administrator and run:

```sql
CREATE DATABASE osticket
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

CREATE USER 'osticket_user'@'localhost'
  IDENTIFIED BY 'REPLACE_WITH_A_STRONG_PASSWORD';

GRANT ALL PRIVILEGES
  ON osticket.*
  TO 'osticket_user'@'localhost';

FLUSH PRIVILEGES;
```

> Do not copy the placeholder password literally. Generate a unique password and store it securely.

## Validate the Database Account

Test the application account:

```powershell
mysql -u osticket_user -p -h localhost osticket
```

Then run:

```sql
SHOW DATABASES;
```

### Why This Matters

Using a dedicated application account follows the principle of **least privilege**. If application credentials are ever compromised, the account is restricted to the application's database rather than granting administrative control over the entire MySQL server.

---

# Part 6 — Install osTicket

## Download osTicket

Download the current **osTicket 1.18.x** release from the official osTicket website.

Extract the archive.

Inside the extracted package, locate the `upload` directory.

Copy its contents into:

```text
C:\inetpub\wwwroot\osTicket
```

The site should resemble:

```text
C:\inetpub\wwwroot\osTicket\
├── api\
├── include\
├── scp\
├── setup\
├── index.php
└── ...
```

---

# Part 7 — Prepare the osTicket Configuration File

Navigate to:

```text
C:\inetpub\wwwroot\osTicket\include
```

Copy or rename:

```text
ost-sampleconfig.php
```

to:

```text
ost-config.php
```

## Temporary Installation Permissions

The osTicket installer needs temporary write access to `ost-config.php`.

Grant the IIS application identity only the permissions necessary to complete installation.

> **Do not grant `Everyone` Full Control.**

After installation, these permissions will be removed.

---

# Part 8 — Run the osTicket Installer

Open:

```text
http://localhost/osTicket
```

The osTicket prerequisite page should appear.

## Resolve Prerequisite Warnings

Do not continue until required prerequisites pass.

If an extension is missing:

1. Open `C:\PHP\php.ini`
2. Enable the appropriate extension
3. Save the file
4. Restart IIS:

```powershell
iisreset
```

5. Refresh the osTicket installer

---

# Part 9 — Complete the Web Installation

When the prerequisite checks pass, click **Continue**.

Configure the help desk.

### Help Desk Information

Example:

```text
Helpdesk Name: CourseCareers Help Desk
Default Email: your lab email address
```

### Administrator Account

Create a unique osTicket administrator account and password.

Do not reuse:

- Your Azure password
- Your Windows administrator password
- Your MySQL root password
- Your database application password

### Database Configuration

Enter:

```text
MySQL Hostname: localhost
MySQL Database: osticket
MySQL Username: osticket_user
MySQL Password: <your generated password>
```

Start the installation.

---

# Part 10 — Post-Installation Security

After the installer reports success:

## Delete the Setup Directory

Delete:

```text
C:\inetpub\wwwroot\osTicket\setup
```

## Restrict ost-config.php

Remove the temporary write permissions that were granted during installation.

The application should no longer have unnecessary write access to:

```text
C:\inetpub\wwwroot\osTicket\include\ost-config.php
```

This file contains sensitive application configuration.

---

# Part 11 — Validate the Installation

## Staff Control Panel

Browse to:

```text
http://localhost/osTicket/scp/login.php
```

Sign in with your osTicket administrator account.

## End-User Portal

Browse to:

```text
http://localhost/osTicket
```

Both pages should load successfully.

---

# Completion Criteria

Before moving on to the next osTicket lab, verify all of the following:

- [ ] Azure VM is deployed
- [ ] RDP access works
- [ ] IIS is installed and running
- [ ] CGI is enabled
- [ ] PHP is installed and registered with IIS
- [ ] Required PHP extensions pass the osTicket prerequisite check
- [ ] MySQL is installed and running
- [ ] `osticket` database exists
- [ ] `osticket_user` exists
- [ ] osTicket does not use MySQL `root` for normal application access
- [ ] osTicket installation completes successfully
- [ ] Staff Control Panel is accessible
- [ ] End-user portal is accessible
- [ ] `setup` directory has been removed
- [ ] `ost-config.php` permissions have been tightened

---

# Troubleshooting

A dedicated troubleshooting guide is included here:

[docs/troubleshooting.md](docs/troubleshooting.md)

Start with evidence before changing settings randomly:

```powershell
Get-Service W3SVC
Get-Service MySQL*
C:\PHP\php.exe -v
C:\PHP\php.exe -m
iisreset
```

---

# Suggested Portfolio Evidence

When using this project in your portfolio, capture screenshots showing:

1. Azure VM Overview page
2. Successful RDP session
3. IIS default site or IIS Manager
4. PHP Manager or `php -v`
5. MySQL service running
6. osTicket prerequisite page with checks passing
7. Successful osTicket installation
8. osTicket Staff Control Panel
9. osTicket end-user portal

Store screenshots in the `images/` directory and reference them from this README.

> Never include passwords, access tokens, private information, or other credentials in screenshots or commits.

---

# Skills Demonstrated

This project demonstrates hands-on exposure to:

- Microsoft Azure administration
- Virtual machine deployment
- Remote administration
- Network Security Groups
- Windows Server administration
- IIS web-server configuration
- PHP application hosting
- MySQL administration
- SQL database/user creation
- Identity and access concepts
- Least privilege
- File permissions
- Web application deployment
- Application troubleshooting
- Technical documentation

---

# Real-World Considerations

This is a training environment.

A production implementation would normally require additional controls such as:

- HTTPS/TLS
- DNS
- Restricted administrative access
- Backups
- Database protection
- Centralized logging
- Vulnerability and patch management
- Secure secrets management
- Monitoring
- Email integration using modern authentication
- High availability or disaster recovery requirements where appropriate

---

# Cleanup and Azure Cost Control

When you are not actively using the lab:

1. Save your work.
2. Sign out of the VM.
3. **Stop/Deallocate** the Azure VM.

Do not delete the VM if you plan to continue with the next osTicket labs.

---

# Next Lab

The next project in the series will focus on **post-installation help desk configuration**, including:

- Roles
- Departments
- Teams
- Agents
- Users
- SLAs
- Help Topics

---

## Author

Created as part of the **CourseCareers Information Technology** hands-on lab series.

This repository is intended for educational and portfolio use.
