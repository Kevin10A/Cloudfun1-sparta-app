Deployment Steps and Automation Levels
--------------------------------------

### Manual Setup

#### Initial Setup:

1.  **Accessing the Virtual Machine:**

    -   Logged into the virtual machine via SSH using secure credentials and the machine's public IP address.

2.  **System Updates:**

    -   Ensured the system was updated and upgraded by running:

        ``` bash
        sudo apt update -y && sudo apt upgrade -y
        ```

3.  **Installing MongoDB:**

    -   Followed MongoDB's official installation guide:

        -   Imported MongoDB's public key and repository.

        -   Installed the MongoDB package using the `apt-get install` command.

        -   Verified the installation by checking the service status with `systemctl status mongod`.

4.  **Configuring MongoDB:**

    -   Edited the MongoDB configuration file located at `/etc/mongod.conf` to allow remote connections:

        -   Replaced `127.0.0.1` with `0.0.0.0` under the `bindIp` setting.

        -   Restarted the MongoDB service to apply changes:

            ```
            sudo systemctl restart mongod
            ```

5.  **Installing Node.js:**

    -   Installed Node.js by adding its official repository and running the setup script:

        ``` bash
        curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
        sudo apt-get install -y nodejs
        ```

6.  **Cloning the Application Repository:**

    -   Used `git clone` to pull the application code from the repository.

    -   Navigated to the appropriate directory to set up the application.

7.  **Setting Up Environment Variables:**

    -   Manually configured the environment variables required by the application, including the database host connection string:

        ``` bash
        export DB_HOST=mongodb://<database-ip>:27017/posts
        ```

8.  **Installing Application Dependencies:**

    -   Ran `npm install` to fetch and install the required Node.js packages for the application.

9.  **Starting the Application:**

    -   Launched the application using Node.js or a process manager like PM2 to keep it running in the background:

        ``` bash 
        node app.js
        ```

* * * * *

### Bash Scripting

#### Automated Setup with Bash Scripts:

1.  **Script Creation:**

    -   Developed separate Bash scripts to handle the setup and configuration of the database and application tiers. These scripts eliminated the need for repetitive manual steps, improving deployment speed and consistency.

2.  **Database Script:**

    -   Automated the installation, configuration, and startup of MongoDB.

    -   Included commands for:

        -   System updates.

        -   Importing MongoDB's repository and installing the package.

        -   Modifying the MongoDB configuration file for remote access.

        -   Enabling and starting the MongoDB service.

3.  **Application Script:**

    -   Automated the setup of the Node.js application, including:

        -   Installing Nginx and configuring it as a reverse proxy.

        -   Installing Node.js and application dependencies.

        -   Cloning the application repository.

        -   Setting up environment variables and starting the application with PM2.

4.  **Benefits of Bash Scripts:**

    -   Reduced deployment time by automating repetitive tasks.

    -   Minimized human error during setup.

    -   Ensured consistency across multiple deployments.

* * * * *


# Automating Two-Tier Deployment

The deployment is fully automated with:

-   **Database Script**: Sets up and configures MongoDB.

-   **App Script**: Configures the application tier with Nginx, Node.js, and the app itself.

-   **User Data**: Runs the app script automatically on instance creation, ensuring immediate deployment.
------------------------------
Bash Scripts
------------
### Sparta App Script
``` bash
#!/bin/bash
# Sparta App Script

# Update packages
sudo apt update -y

# Upgrade packages (completely non-interactive)
sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y

# Install Nginx
sudo DEBIAN_FRONTEND=noninteractive apt-get install nginx -y

# Configure Nginx as a reverse proxy
sudo sed -i 's|try_files.*;|proxy_pass http://localhost:3000/;|' /etc/nginx/sites-available/default

# Restart Nginx to apply changes
sudo systemctl restart nginx

# Download Node.js setup script
curl -fsSL https://deb.nodesource.com/setup_20.x -o setup_nodejs.sh

# Run Node.js setup script
sudo DEBIAN_FRONTEND=noninteractive bash setup_nodejs.sh

# Install Node.js
sudo DEBIAN_FRONTEND=noninteractive apt-get install nodejs -y

# Clone the Git repository
git clone https://github.com/daraymonsta/tech201-sparta-app repo

# Navigate to the app directory
cd repo/app

# Set MongoDB environment variable (update IP as needed)
export DB_HOST=mongodb://10.0.3.4:27017/posts

# Install PM2 globally
sudo npm install -g pm2

# Install app dependencies
npm install

# Populate the database
node seeds/seed.js

# Start the app with PM2
pm2 start app.js --name sparta-app

# Save PM2 process list
pm2 save

# Ensure PM2 starts on boot
pm2 startup

# Check versions of Node.js and npm
node -v
npm -v
```



### Database Script
``` bash
#!/bin/bash
# Database Script

# update the system
sudo apt update -y
echo "System packages updated."

# upgrade exclude debian
sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y

# Install MongoDB dependencies
sudo DEBIAN_FRONTEND=noninteractive apt-get install gnupg curl -y
echo "MongoDB installation dependencies installed."

# MongoDB installation steps
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
    sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
echo "MongoDB public GPG key imported."

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

sudo apt-get update -y
echo "Package list updated."

# Install MongoDB packages
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y mongodb-org=7.0.6 mongodb-org-database=7.0.6 mongodb-org-server=7.0.6 mongodb-mongosh mongodb-org-mongos=7.0.6 mongodb-org-tools=7.0.6
echo "MongoDB installed."

# MongoDB version protection
echo "mongodb-org hold" | sudo dpkg --set-selections
echo "mongodb-org-database hold" | sudo dpkg --set-selections
echo "mongodb-org-server hold" | sudo dpkg --set-selections
echo "mongodb-mongosh hold" | sudo dpkg --set-selections
echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
echo "mongodb-org-tools hold" | sudo dpkg --set-selections
echo "MongoDB version control enabled."

# start MongoDB service
sudo systemctl start mongod
echo "MongoDB service started."

# Check the status of MongoDB service
sudo systemctl status mongod
echo "Checked MongoDB service status."

# Edit MongoDB config file to allow connections from other IPs
sudo sed -i 's/127.0.0.1/0.0.0.0/' /etc/mongod.conf
echo "MongoDB config updated to allow remote connections."

# restart MongoDB
sudo systemctl restart mongod
echo "MongoDB service restarted."

# Check MongoDB service status again
sudo systemctl status mongod
echo "Checked MongoDB service status again."

# enable MongoDB to start on boot
sudo systemctl enable mongod
echo "MongoDB enabled on startup."
```
---

## User Data

This section provides information on user-specific configurations, environment variables, and other customization options required for deploying the Sparta App and Database.

## What is a User Data Script?

A **User Data Script** is a script that runs automatically when a new virtual machine (VM) or instance is launched in a cloud environment, such as AWS, Azure, or Google Cloud. It allows you to automate the initial setup of the VM, configure software, and perform other initialization tasks without needing manual intervention.

User Data scripts are typically used for:

- Installing packages or software
- Configuring system settings
- Setting up environment variables
- Starting services or applications

## How User Data Works

When you create a new VM or instance, you can specify a **User Data** field. This field accepts scripts that are executed upon the instance's first boot. These scripts are typically written in **Bash** (for Linux-based systems) or **PowerShell** (for Windows-based systems).

### MongoDB Environment Variable

The `DB_HOST` environment variable in the Sparta App script must point to the correct MongoDB server address to establish a connection between the app and the database.  

- **Default Value:**  
  `export DB_HOST=mongodb://10.0.3.4:27017/posts`
  
- **How to Customize:**  
  Replace `10.0.3.4` with the IP address or hostname of your MongoDB server. For instance:  
  ```bash
  export DB_HOST=mongodb://<your-mongodb-ip>:27017/posts
  ```

---

## Automation with Custom VM Images

Custom VM images offer a powerful method to streamline deployment by creating virtual machines (VMs) pre-configured with the necessary software, settings, and dependencies. This approach eliminates the repetitive process of manually configuring each VM, ensuring consistency and efficiency across multiple deployments.

### Why Use Custom VM Images?

Using custom VM images allows you to:  
1. **Save Time:** Pre-installed dependencies reduce the time required to configure new VMs.  
2. **Ensure Consistency:** All VMs based on the image will have identical configurations, minimizing discrepancies.  
3. **Enable Scalability:** Quickly deploy multiple VMs for load balancing, scaling, or redundancy.

---

### Our Approach: Marketplace Image and Custom Image Process

To create our app VM, we began with a **free marketplace image** for Ubuntu. This allowed us to pre-install all required dependencies and configure the VM for the application.  

#### Why a Marketplace Image?  
The original image we used was a free Ubuntu image available in the Azure Marketplace. Using a marketplace image provided the following benefits:  
- **Pre-tested Baseline:** Marketplace images are validated by their providers (Canonical, in this case) and are guaranteed to work on Azure.  
- **Cost Efficiency:** As a free resource, it allowed us to build our custom image without additional costs.  

#### Plan Information Requirement  
When using a custom image derived from a marketplace image, you must include "plan information" to credit the original image's source. This ensures that the original provider (Canonical) receives proper acknowledgment. Without this, deploying new VMs using the custom image will result in an error.

---

### Process for Creating the Custom Image

1. **Initial Setup by Instructor Ramon:**  
   - Ramon downloaded an **Ubuntu Virtual Hard Disk (VHD)** from the official Ubuntu website.  
   - He **uncompressed the VHD twice** to extract its contents.  
   - The uncompressed VHD was uploaded to **Azure Blob Storage**, where it served as the base for a custom VM.

2. **Creating the First Custom VM:**  
   - Ramon created a VM using the uploaded Ubuntu VHD.  
   - This VM was configured with all necessary components:  
     - **Nginx** for serving web requests.  
     - **Node.js** for running the application.  
     - A reverse proxy setup.  
     - The application folder located at `/repo/app`.

3. **Creating the Custom App Image:**  
   - Using the working app VM, a **generalized image** was created, named `ramon-official-ubuntu2204-clean-image`.  
   - This image served as the foundation for future VMs.  

---

### Steps to Create and Test Custom App Images

#### 1. Create a Custom App Image  
   Using the working app VM, a new generalized image was created with all app dependencies pre-installed. For example:  
   - Name: `cloudfun1-firstname-ready-to-run-app-image`.  

#### 2. Test the Custom Image  
   - Create a new app VM from the custom app image.  
   - During VM creation, include a **Bash script** in the user data to automatically start the application.

##### Bash Script Example  
```bash
#!/bin/bash
cd /repo/app
node app.js
```
* * * * *
---

## Troubleshooting and Blockers

During the deployment process, several blockers and issues were encountered. Below is a detailed explanation of each issue, its impact, and the solutions implemented to resolve them.

---

### Blocker 1: MongoDB Remote Connections  

**Issue:**  
Unable to connect to MongoDB remotely from application VMs. By default, MongoDB restricts connections to `127.0.0.1`, limiting access to localhost only.  

**Impact:**  
This restriction prevented the application from accessing the MongoDB database hosted on a separate VM.  

**Solution:**  
- Edited the MongoDB configuration file (`/etc/mongod.conf`) to allow connections from all IP addresses by replacing the following line:  
  ```bash
  bindIp: 127.0.0.1
  ```
* * * * *

### Blocker 2: Node.js Compatibility

**Issue:**\
The initial version of Node.js installed on the VM was outdated, causing compatibility issues with application dependencies.

**Impact:**\
Dependency installation (`npm install`) failed, blocking the application's setup process.

**Solution:**

-   Downloaded and executed the official Node.js setup script to install the latest version:
bash
Copy code

``` bash
curl -fsSL https://deb.nodesource.com/setup_20.x -o setup_nodejs.sh
sudo bash setup_nodejs.sh
sudo apt-get install -y nodejs
```
- Verified the Node.js and npm versions:
``` bash
node -v
npm -v
```
* * * * *

### Blocker 3: PM2 Not Persisting on Reboot

**Issue:**\
PM2 (Process Manager 2) processes managing the application did not restart automatically after a VM reboot.

**Impact:**\
Application downtime occurred every time the VM was restarted, requiring manual intervention to start the processes.

**Solution:**

-   Configured PM2 to persist across reboots by running the following commands:
``` bash
pm2 startup
pm2 save
```
* * * * *

### Blocker 4: Reverse Proxy Not Working

**Issue:**\
The reverse proxy configuration in Nginx was correctly set, but HTTP traffic (port 80) was not accessible due to missing inbound networking rules.

**Impact:**\
The application was unreachable over the web, blocking external access to the service.

**Solution:**

-   Returned to the VM's **Overview** page in Azure.
-   Navigated to the **Networking** tab and added a new inbound rule to allow HTTP traffic on port 80:
    1.  **Source:** Any
    2.  **Protocol:** TCP
    3.  **Port:** 80
    4.  **Action:** Allow

* * * * *

### Lessons Learned

-   **Proactive Configuration Checks:** Ensuring configurations like networking rules and service settings are reviewed during deployment reduces troubleshooting time.
-   **Automation Is Key:** Automating common setups like PM2 persistence and dependency installation minimizes human error.
-   **Monitoring Tools:** Implementing monitoring for services such as MongoDB and PM2 can alert the team to issues earlier.
-   
* * * * *
# Screenshots
![Automation Levels] ![Images/Automation Levels.jpg](<Images/Automation Levels.jpg>)


![Bash Database]![Images/Bash Database Script.png](<Images/Bash Database Script.png>)


![Custom Image Diagram]![Images/Custom Image Diagram.jpg](<Images/Custom Image Diagram.jpg>)

![Post Sites]![Images/Posts Site.png](<Images/Posts Site.png>)

![Image Reverse Proxy Problem]![Images/Reverse Proxy Problem.png](<Images/Reverse Proxy Problem.png>)

![User Data Script]![Images/User Data Script .png](<Images/User Data Script .png>)

![User Image]![Images/User-Image Script.png](<Images/User-Image Script.png>)

![VM Image Script]![Images/VM Image Script.png](<Images/Vm-Image-Script.png>)

![Working Site]![Images/Working Site.PNG>](<Images/Working Site.PNG>)