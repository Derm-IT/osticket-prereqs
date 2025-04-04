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

### 1. Azure Virtual Machine Setup (for more detailed steps go to [here](https://github.com/Derm-IT/azure-network-protocols#part-1-virtual-machine-creation))

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
    *  Connect to the public ip of osticket-vm found here
    *  ![image](https://github.com/user-attachments/assets/d91fce62-9241-4514-89da-a95bc75a5329)
    *  Sign in as user you just created
    *  ![image](https://github.com/user-attachments/assets/b878a89d-3799-40c4-8cc4-774638a7ee46)

### 2. Initial VM Preparation & Prerequisite Staging

*  Obtain Installation Files.
    * Download [osTicket-Installation-Files.zip](https://drive.google.com/uc?export=download&id=1b3RBkXTLNGXbibeMuAynkfzdBC1NnqaD) on your VM.
    * Unzip the contents to the VM's Desktop.
    * Ensure the resulting folder is named osTicket-Installation-Files
    * **Reason:** Consolidating all necessary installers and files simplifies the setup process.
    * Folder should look like this on the Desktop
    * ![image](https://github.com/user-attachments/assets/2678af79-1429-4544-a256-c376a7770532)
      
*  Install/Enable Internet Information Services (IIS) to host and serve osTicket
    *  Open Control Panel -> Programs -> Programs and Features
    *  Click "Turn Windows features on or off".
    *  Check Internet Information Services
    * **Crucial Component:** Ensure **CGI** is enabled under World Wide Web Services -> Application Development Features.
    * CGI (Common Gateway Interface) is required for IIS to execute PHP scripts, which osTicket is built upon.
    * ![image](https://github.com/user-attachments/assets/a9f3cc10-f379-4339-aac1-ee87e17f8f05)
    * Enter 127.0.0.1 (loopback address) to check if its running
    * ![image](https://github.com/user-attachments/assets/b8934046-c354-4fcd-8e89-1036ae492a0c)
    * Should see IIS default webpage

*  Install IIS Modules/Tools (from osTicket-Installation-Files folder).
    * Install **PHP Manager for IIS** (PHPManagerForIIS_V1.5.0.msi).
        * Agree to everything
        * ![image](https://github.com/user-attachments/assets/dd6a5396-5c45-4fec-8f85-eb0fef4f65a9)
        * Why? Provides a user-friendly interface within IIS Manager to manage PHP versions, settings, and extensions.
    * Install **Rewrite Module** (rewrite_amd64_en-US.msi).
        * Agree and finish
        * ![image](https://github.com/user-attachments/assets/704164a5-bd68-403c-a5f3-5650c9116a11)
        * osTicket uses URL rewriting for cleaner URLs and proper routing. This module enables that functionality within IIS.
*  Create PHP Directory.
    * Create the folder C:\PHP.
    * Providing a dedicated, standard location for the PHP installation files.
*  Install PHP.
    * Unzip php-7.3.8-nts-Win32-VC15-x86.zip (from osTicket-Installation-Files) into the C:\PHP directory.
    * ![image](https://github.com/user-attachments/assets/d5b4cb63-9b98-4be2-b9fe-6fae6e70c8b6)
*  Install Visual C++ Redistributable (from osTicket-Installation-Files folder).
    * Install VC_redist.x86.exe.
    * PHP for Windows relies on the Microsoft Visual C++ runtime libraries. This installer provides those necessary DLLs.

### 3. Database Setup: MySQL

*  Install MySQL Server.
    * Run mysql-5.5.62-win32.msi (from osTicket-Installation-Files).
    * **Setup Type:** Choose Typical.
    * **Post-Install:** Ensure "Launch the MySQL Instance Configuration Wizard" is checked and click Finish.
    * **Configuration Wizard:**
        * Select Standard Configuration.
        * Keep defaults (Install as Windows Service, etc.).
        * Set MySQL root **Password:** root. (**Security Warning:** Use a strong, unique password in production!).
        * ![image](https://github.com/user-attachments/assets/f91028b4-c191-4c65-bfe7-75944862cd42)
        * Execute the configuration.
    * ? osTicket requires a database to store ticket data, user information, configurations, etc. MySQL is a popular open-source choice compatible with osTicket.

### 4. IIS Configuration for PHP

*  Register PHP with IIS.
    * Open **IIS Manager** (Run as Administrator).
    * Select the server node (top level) in the left pane.
    * Double-click **PHP Manager** in the center pane.
    * ![image](https://github.com/user-attachments/assets/4217fadf-bdfd-4787-b508-638525271849)
    * Click "**Register new PHP version**".
    * Browse to and select the PHP executable: C:\PHP\php-cgi.exe.
    * ![image](https://github.com/user-attachments/assets/a5d16566-902f-43e9-8d60-97b3a57fea23)
    * Click OK.
    * This tells IIS where to find the PHP interpreter and how to execute PHP files using the CGI protocol 
*  Reload IIS.
    * In IIS Manager, select the server node (osticket-vm) on the left
    * In the right Actions pane, click **Stop**.
    * Once stopped, click **Start**.
    * Ensures that all configuration changes, including the PHP registration, are loaded and applied by the webserver.

### 5. osTicket Application Installation

*  Deploy osTicket Files.
    * From osTicket-Installation-Files, unzip osTicket-v1.15.8.zip (can unzip into same folder)
    * Copy the upload folder from the unzipped contents into c:\inetpub\wwwroot.
    * ![image](https://github.com/user-attachments/assets/902df23f-d8f3-4644-aa12-6debd9341986)
    * Rename the upliad folder (c:\inetpub\wwwroot\upload) to osTicket (c:\inetpub\wwwroot\osTicket).
    * Make sure it is exactly `osTicket`
    * **Reason:** c:\inetpub\wwwroot is the default web root directory for IIS. Placing the osTicket application files here makes them accessible via the webserver. Renaming upload to osTicket provides a clean URL path (http://localhost/osTicket).
*  Reload IIS.
    * Stop and Start the IIS server again via IIS Manager.
    * Ensures IIS recognizes the new osTicket application folder.

### 6. osTicket Pre-Setup Configuration (Web & Filesystem)

*  Initiate Web Setup & Identify Missing Extensions.
    * In IIS Manager, navigate to Sites -> Default Web Site -> osTicket.
    * In the right Actions pane, under "Browse Website", click "**Browse *:80 (http)**".
    * Your browser should open to the osTicket setup page (http://localhost/osTicket/setup/).
    * ![image](https://github.com/user-attachments/assets/b2f9af9a-1ea5-4682-b1a7-63f63d0e1ef3)
    * **Observe:** The setup page will likely indicate that some required PHP extensions are missing or disabled.
    * ![image](https://github.com/user-attachments/assets/4d2c7035-ef8a-4df1-81eb-ee1f5fa6dc13)
*  Enable Required PHP Extensions.
    * Go back to IIS Manager -> Sites -> Default Web Site -> osTicket.
    * Double-click **PHP Manager**.
    * Click "**Enable or disable an extension**".
    * Locate and **Enable** the following extensions (right click each and click enable:
        * php_imap.dll (For interacting with mailboxes via IMAP protocol).
        * php_intl.dll (For internationalization features).
        * php_opcache.dll (Improves PHP performance by caching precompiled script bytecode).
        * ![image](https://github.com/user-attachments/assets/a99f4d31-fed6-4611-823a-dc653faaf65f)
    * Go back to your browser and **refresh** the osTicket setup page. The extension warnings should disappear.
    * ![image](https://github.com/user-attachments/assets/1ce570ca-fa7e-4560-a386-92fc9f344d0b)
*  Prepare osTicket Configuration File.
    * Navigate to C:\inetpub\wwwroot\osTicket\include\.
    * Scroll down to ost-sampleconfig.php
    * ![image](https://github.com/user-attachments/assets/e2cb2a43-ff1c-443f-b5e5-55ca81958a31)
    * Rename ost-sampleconfig.php to ost-config.php.
    * ![image](https://github.com/user-attachments/assets/e6f407c0-08c1-416b-a8c5-a25c00330a25)
    * osTicket reads its core database connection settings from ost-config.php. The setup process needs to write to this file.
*  Set Permissions for Setup.
    * Right-click on C:\inetpub\wwwroot\osTicket\include\ost-config.php -> Properties -> Security -> Advanced.
    * Click **Disable inheritance**, choose "Remove all inherited permissions...".
    * Click **Add**, click "Select a principal", type Everyone, click OK.
    * Grant Everyone **Full control** permissions.(Bad Security. Usually only give access to IIS users (IIS_IUSRS) and not full control but for this project its simpler to give it everyone as we won't be using it in production)
    * Click OK on all dialogs.
    * ![image](https://github.com/user-attachments/assets/93d7fe27-20b1-4b18-8a27-cbb6ed854a09)
    * The web setup process (running under the IIS application pool identity) needs permission to write the database connection details into ost-config.php. This permissive setting is temporary for setup.
    * **Security Tip:** These permissions will be restricted later after setup is complete.

### 7. Database Setup (HeidiSQL)

*  Install Database Management Tool.
    * From osTicket-Installation-Files, install **HeidiSQL**.
    * Agree to everything 
    * HeidiSQL is a lightweight graphical tool for managing MySQL databases, making tasks like creating databases easier than using the command line.
*  Create the osTicket Database.
    * Open HeidiSQL.
    * Create a **New** session.
        * **Hostname/IP:** 127.0.0.1 (or localhost)
        * **User:** root
        * **Password:** root (SAME AS WHEN WE SETUP SQL)
        * **Port:** 3306 (default)
        * ![image](https://github.com/user-attachments/assets/4af7567b-f010-4d2a-95cf-6d2979adbe66)
    * Click **Open** to connect
    * Right-click Unnamed in the left pane (on the server connection) -> Create new -> Database.
    * Enter the database name: osTicket (make sure its spelled exact!!!)
    * ![image](https://github.com/user-attachments/assets/feccc440-caeb-47d4-aca9-bc8e9d63a433)
    * Click OK, should see it here
    * ![image](https://github.com/user-attachments/assets/914e1fe7-e181-406e-9b34-0707582d9340)
    * osTicket needs a dedicated database to store its data and this is that

### 8. Finalizing osTicket Web Setup

*  Complete Installation via Browser.
    * Go back to the osTicket setup page in your browser (or refresh it).
    * Click **Continue** past the prerequisites check.
    * **System Settings:**
        * Enter a **Helpdesk Name** (e.g., "My IT Support").
        * Enter the **Default System Email** (needs to be a valid email address).
    * **Admin User:**
        * Configure the initial administrator account details (First Name, Last Name, Email (has to be different than above), Username, Password).
        * Easiest to use same user and password as the VM (for practice purposes)
    * **Database Settings:**
        * **MySQL Database:** osTicket (the database created earlier).
        * **MySQL Username:** root
        * **MySQL Password:** root
    * Click **Install Now!**.
    * ![image](https://github.com/user-attachments/assets/b24b1736-2413-4ad2-ad99-af820783ea12)
*  Wait for the installation process to complete. You should see a "Congratulations!" message.
*  ![image](https://github.com/user-attachments/assets/8141327e-2461-4331-b0c8-44c8f8e6423f)

### 9. Post-Installation Cleanup & Security

*  Remove Setup Directory.
    * Delete the folder C:\inetpub\wwwroot\osTicket\setup\.
    * This directory is only needed for installation and leaving it accessible is a major security risk.
    * Set Permissions to “Read” only: C:\inetpub\wwwroot\osTicket\include\ost-config.php
    * ![image](https://github.com/user-attachments/assets/c7055fe7-42ad-4e81-b4b7-20faf96cf18d)
