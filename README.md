# osTicket: Prerequisites and Installation

## Overview

This repository documents the process of installing and configuring the osTicket helpdesk system on an Azure Windows 10 Virtual Machine. This setup includes integrating osTicket with Internet Information Services (IIS), PHP, and MySQL, providing a comprehensive guide for deploying osTicket in a Windows server environment.

## Objective

To deploy osTicket on a Windows 10 VM in Azure, creating a functional helpdesk system for managing support requests.

## Context

This project demonstrates the skills needed for IT roles involving system administration, application support, and web hosting, particularly in deploying web applications on Windows servers.

## Prerequisites

* An active Azure subscription.
* An Azure Resource Group
* Basic understanding of Windows server administration.
* Remote Desktop Protocol (RDP) client.

## Steps

### 1. Azure Virtual Machine Setup (for more detailed steps go to 

* Create an Azure Virtual Machine.
    * **Image:** Windows 10 (Check box for licensing!)
    * **Size/vCPUs:** 2 vCPUs
    * **VM Name:** osticket-vm
    * **Region:** same as Resource Group
    * **Admin Username:** adminuser (any username you want)
    * **Admin Password:** ***************
    * Leave everything else default and create it
    * ![image](https://github.com/user-attachments/assets/c2c9cc4f-6c78-4ebf-9276-723275f0415d)
* Connect to the VM.
    *  Open Remote Desktop Connect (RDP).
    *  Connect to the public ip of osticket-vm
    *  Sign in as user you just created

### 2. Initial VM Preparation & Prerequisite Staging

*  Obtain Installation Files.
    * Download osTicket-Installation-Files.zip.
    * Unzip the contents to the VM's Desktop.
    * Ensure the resulting folder is named osTicket-Installation-Files.
    * **Reason:** Consolidating all necessary installers and files simplifies the setup process.
*  Install/Enable Internet Information Services (IIS).
    *  Use "Turn Windows features on or off".
    * **Crucial Component:** Ensure **CGI** is enabled under World Wide Web Services -> Application Development Features.
    * **Reason:** CGI (Common Gateway Interface) is required for IIS to execute PHP scripts, which osTicket is built upon.
*  Install IIS Modules/Tools (from osTicket-Installation-Files folder).
    * Install **PHP Manager for IIS** (PHPManagerForIIS_V1.5.0.msi).
        * **Reason:** Provides a user-friendly interface within IIS Manager to manage PHP versions, settings, and extensions.
    * Install **Rewrite Module** (rewrite_amd64_en-US.msi).
        * **Reason:** osTicket uses URL rewriting for cleaner URLs and proper routing. This module enables that functionality within IIS.
*  Create PHP Directory.
    * Create the folder C:\PHP.
    * **Reason:** Provides a dedicated, standard location for the PHP installation files.
*  Install Visual C++ Redistributable (from osTicket-Installation-Files folder).
    * Install VC_redist.x86.exe.
    * **Reason:** PHP for Windows relies on the Microsoft Visual C++ runtime libraries. This installer provides those necessary DLLs.

### 3. Dependency Installation: PHP & MySQL

*  Install PHP.
    * Unzip php-7.3.8-nts-Win32-VC15-x86.zip (from osTicket-Installation-Files) into the C:\PHP directory.
    * **Note:** This is PHP version 7.3.8, Non-Thread Safe (NTS), for x86 architecture, compiled with VC15. NTS is generally recommended when using IIS with FastCGI.
*  Install MySQL Server.
    * Run mysql-5.5.62-win32.msi (from osTicket-Installation-Files).
    * **Setup Type:** Choose Typical.
    * **Post-Install:** Ensure "Launch the MySQL Instance Configuration Wizard" is checked and click Finish.
    * **Configuration Wizard:**
        * Select Standard Configuration.
        * Keep defaults (Install as Windows Service, etc.).
        * Set MySQL root **Password:** root. (**Security Warning:** Use a strong, unique password in production!).
        * Execute the configuration.
    * **Reason:** osTicket requires a database to store ticket data, user information, configurations, etc. MySQL is a popular open-source choice compatible with osTicket.

### 4. IIS Configuration for PHP

*  Register PHP with IIS.
    * Open **IIS Manager** (Run as Administrator).
    * Select the server node (top level) in the left pane.
    * Double-click **PHP Manager** in the center pane.
    * Click "**Register new PHP version**".
    * Browse to and select the PHP executable: C:\PHP\php-cgi.exe.
    * Click OK.
    * **Reason:** This tells IIS where to find the PHP interpreter and how to execute PHP files using the CGI protocol (specifically, FastCGI is typically configured by PHP Manager).
*  Reload IIS.
    * In IIS Manager, select the server node.
    * In the right Actions pane, click **Stop**.
    * Once stopped, click **Start**.
    * **Reason:** Ensures that all configuration changes, including the PHP registration, are loaded and applied by the webserver.

### 5. osTicket Application Installation

*  Deploy osTicket Files.
    * From osTicket-Installation-Files, unzip osTicket-v1.15.8.zip.
    * Copy the upload folder from the unzipped contents into c:\inetpub\wwwroot.
    * Rename the folder c:\inetpub\wwwroot\upload to c:\inetpub\wwwroot\osTicket.
    * **Reason:** c:\inetpub\wwwroot is the default web root directory for IIS. Placing the osTicket application files here makes them accessible via the webserver. Renaming upload to osTicket provides a clean URL path (http://localhost/osTicket).
*  Reload IIS.
    * Stop and Start the IIS server again via IIS Manager.
    * **Reason:** Ensures IIS recognizes the new osTicket application folder.

### 6. osTicket Pre-Setup Configuration (Web & Filesystem)

*  Initiate Web Setup & Identify Missing Extensions.
    * In IIS Manager, navigate to Sites -> Default Web Site -> osTicket.
    * In the right Actions pane, under "Browse Website", click "**Browse *:80 (http)**".
    * Your browser should open to the osTicket setup page (http://localhost/osTicket/setup/).
    * **Observe:** The setup page will likely indicate that some required PHP extensions are missing or disabled.
*  Enable Required PHP Extensions.
    * Go back to IIS Manager -> Sites -> Default Web Site -> osTicket.
    * Double-click **PHP Manager**.
    * Click "**Enable or disable an extension**".
    * Locate and **Enable** the following extensions:
        * php_imap.dll (For interacting with mailboxes via IMAP protocol).
        * php_intl.dll (For internationalization features).
        * php_opcache.dll (Improves PHP performance by caching precompiled script bytecode).
    * Go back to your browser and **refresh** the osTicket setup page. The extension warnings should disappear.
*  Prepare osTicket Configuration File.
    * Navigate to C:\inetpub\wwwroot\osTicket\include\.
    * Rename ost-sampleconfig.php to ost-config.php.
    * **Reason:** osTicket reads its core database connection settings from ost-config.php. The setup process needs to write to this file.
*  Set Permissions for Setup.
    * Right-click on C:\inetpub\wwwroot\osTicket\include\ost-config.php -> Properties -> Security -> Advanced.
    * Click **Disable inheritance**, choose "Remove all inherited permissions...".
    * Click **Add**, click "Select a principal", type Everyone, click OK.
    * Grant Everyone **Full control** permissions.
    * Click OK on all dialogs.
    * **Reason:** The web setup process (running under the IIS application pool identity) needs permission to write the database connection details into ost-config.php. This permissive setting is temporary for setup.
    * **Security Tip:** These permissions will be restricted later after setup is complete.

### 7. Database Setup (HeidiSQL)

*  Install Database Management Tool.
    * From osTicket-Installation-Files, install **HeidiSQL**.
    * **Reason:** HeidiSQL is a lightweight graphical tool for managing MySQL databases, making tasks like creating databases easier than using the command line.
*  Create the osTicket Database.
    * Open HeidiSQL.
    * Create a **New** session.
        * **Hostname/IP:** 127.0.0.1 (or localhost)
        * **User:** root
        * **Password:** root
        * **Port:** 3306 (default)
    * Click **Open** to connect
    * Right-click in the left pane (on the server connection) -> Create new -> Database.
    * Enter the database name: osTicket.
    * Click OK.
    * **Reason:** osTicket needs a dedicated database to store its data.

### 8. Finalizing osTicket Web Setup

*  Complete Installation via Browser.
    * Go back to the osTicket setup page in your browser (or refresh it).
    * Click **Continue** past the prerequisites check.
    * **System Settings:**
        * Enter a **Helpdesk Name** (e.g., "My IT Support").
        * Enter the **Default System Email** (needs to be a valid email address).
    * **Admin User:**
        * Configure the initial administrator account details (First Name, Last Name, Email, Username, Password).
    * **Database Settings:**
        * **MySQL Database:** osTicket (the database created earlier).
        * **MySQL Username:** root
        * **MySQL Password:** root
    * Click **Install Now!**.
* **Verification:** Wait for the installation process to complete. You should see a "Congratulations!" message.

### 9. Post-Installation Cleanup & Security

*  Remove Setup Directory.
    * Delete the folder C:\inetpub\wwwroot\osTicket\setup\.
    * **Reason:** This directory is only needed for installation and leaving it accessible is a major security risk.
