# 2420 Assignment 3 part 2

Hey new user, this tutorial will help you set up an HTML generator that runs everyday at 5:00 am. This will be ran on 2 different arch linux servers which are distributed by a load balancer which is set up in digital ocean. These websites will be served by nginx and a server block configuration set up. All the necessary files are found in this repo, all you need to do is follow this tutorial to configure your Arch Linux to run the files for success!

Before we start setting up the necessary files, we will need to have 2 droplets up and running on the same project. Find the most recent qcow arch linux image in the arch wiki (should be 450mb size) and download it. Also ensure you have configured them using an SSH key pair which you can access through your terminal.

Once made, complete the following steps on both of your droplets.

## 1. Creating a new System User (Webgen)

For this setup, you will need to create a new user named "webgen" for which the files have been geared to execute towards. ***They have been written this way to ensure security (limiting access in case of security breach) and efficiency (blank slate solely intended for these setup files) are met for the execution of this tutorial.***

This user will have a home directory set to /var/lib/webgen and will not have a login shell (since it will not be used for interactive logins). 

To execute, run this command: 
```
sudo useradd --system -d /var/lib/webgen -s /usr/sbin/nologin webgen
```
- --system: Creates a system user
- -d /var/lib/webgen: Sets home directory for the user
- -s /usr/sbin/nologin: Prevents user from logging in through command line

Next we will need to create the necessary subdirectories for files we will be integrating later. 
We will need to make subdirectories for Bin and HTML, we can do this with the following commands: 
```
sudo mkdir /var/lib/webgen/HTML
sudo mkdir /var/lib/webgen/bin
```
Additionally, we will need to make this new user the owner of this directory. 

To execute, run this command:
```
sudo chown -R webgen:webgen /var/lib/webgen
```
- -R: Allows Ownership for all the directories under webgen

## 2. Setting up Necessary Files

The current repository you are in includes all the necessary files. To access these files onto your local machine, execute this command:
```
git clone https://github.com/philiflo/2420-as3.git
```
### generate_index

The generate_index script is responsible for generating the html template that will be injected into the index.html file, which we will be calling to create our website. 

To move the file to its proper location, execute the following command inside the 2420-as3 directory:
```
sudo mv generate_index /var/lib/webgen/bin/
```
Once executed, make sure the script is executable by running:
```
sudo chmod +x /var/lib/webgen/bin/generate_index
```
### generate-index.service and generate-index.timer

The generate-index.service and generate-index.timer file are responsible for the execution of this process daily at 5am. Generate-index.service defines what action must be executed (running the script inside generate_index), while generate-index.timer defines the time in which the file must be executed. 

To move the file to its proper location, execute the following command inside the 2420-as3 directory:
```
sudo mv generate-index.service /etc/systemd/system/
```
Once executed, make sure the script is executable by running:
```
sudo chmod +x /var/lib/webgen/bin/generate_index
```
Once done, reload systemd through: 
```
sudo systemctl daemon-reload
```
Then enable the timer through:
```
sudo systemctl enable --now generate-index.timer
```
### nginx.conf and service-block.conf

Nginx.conf and service-block.conf are necessary to configure nginx for the use of the site we are creating. ***We have seperated the service block into its own file to keep configuration in the nginx.conf file specific to nginx global configuration***

To locate these file in their necessary location, execute the following commands

Nginx.conf:
```
sudo mv nginx.conf /etc/nginx/nginx.conf
```
Create the necessary subdirectories - 
```
sudo mkdir /etc/nginx/sites-available
sudo mkdir /etc/nginx/sites-enabled
```
Then move service-block.conf to the available sites - 
```
sudo mv server-block.conf /etc/nginx/sites-available/
```
Once here, you will need to create a symbolic link to the server-block.conf file which is titled sites-enabled. This will allow the syntax inside the nginx.conf file to connect to the server-block.conf file. Execute this command:
```
sudo ln -s /etc/nginx/sites-available/server-block.conf /etc/nginx/sites-enabled/
```

## 3. Setting up Personal Firewall (UFW)

Once all the files are setup, we can setup our firewall using UFW (uncomplicated firewall). This will effectively create security measures by managing user traffic on our server. 

Begin by installing UFW through the following command: 
```
sudo pacman -S ufw
```
- pacman is the package manager that Arch Linux utilizes, for other Linux distributors, utilize the necessary package manager to ensure a complete installation is executed.

Next, enable HTTPS and SSH on our firewall to allow these traffic commanded by these services. If SSH isn't enabled on your local server before your firewall is enabled, you will effectively block access to your server from your SSH, basically kicking you out from your own server. 

Execute the following commands:
```
sudo ufw allow ssh
sudo ufw allow http
```
To enable your firewall, execute the following command: 
```
sudo systemctl enable --now ufw.service
```
Next we want to enable ssh rate limiting to combat against brute force attacks, a common method of security breaching. 

We can do this through the following command: 
```
sudo ufw limit ssh
```
To allow port 80 to send traffic through our firewall, (which will allow our load balancer to connect to our droplets), execute the following:
```
sudo udw allow 80
```

## 4. Setting Up a Load Balancer

Once here, we can start the configuration of our load balancer. Most of the configuration will be inside the digital ocean interface.


## 5. Execution and Troubleshooting

Once all steps have been executed, we can begin testing to see if all files have been integrated correctly. 

First lets check if the website executes correctly:

***Check if nginx files are executing properly -***
```
sudo nginx -t
sudo systemctl reload nginx
```
In a new browser tab, paste the following to ensure the website has the correct information -

http://164.90.146.179/

This ip address is defined in the service-block.conf file as the ip for this website. The website will include system information such as the version of your OS and the current date if it has been executed correctly. 

***We can then test if the timer and service files have been executed properly with the following commands:***

generate_index.timer - 
```
sudo systemctl status generate-index.timer
```
Look to see if the output states 'active' in green or 'failed' in red.

For generate_index.service, we must first run the script manually if it isn't 5am to see if it runs correctly - 
```
sudo systemctl start generate-index.service
```
Then -
```
sudo systemctl status generate-index.service
```
We can check more specific journal logs through: 
```
sudo journalctl -u generate-index.service
sudo journalctl -u generate-index.timer
```
***Lastly, we can test to see if our firewall is up and running through:***
```
sudo ufw status verbose
```
Expect the following output: 

```
To                         Action      From
--                         ------      ----
80/tcp                     ALLOW IN    Anywhere
22/tcp                     ALLOW IN    Anywhere
80/tcp (v6)                ALLOW IN    Anywhere (v6)
22/tcp (v6)                ALLOW IN    Anywhere (v6)
```

Once you have ensured all steps have been properly troubleshot, you will have successfully completed this tutorial!
