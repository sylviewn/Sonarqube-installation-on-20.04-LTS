# SonarQube Installation on Ubuntu 20.04 LTS server
Step to install Sonarqube on Ubuntu 20.04 LTS

I. Install Java OpenJDK
Java is one of the requirements to install and set up SonarQube on Ubuntu 20.04 or 18.04 and its based operating systems.

Run Ubuntu system update: sudo apt update
Install Java OpenJDK : sudo apt install openjdk-11-jdk
 
I-2 Increase the Virtual memory

sudo sysctl -w vm.max_map_count=524288
sudo sysctl -w fs.file-max=131072
ulimit -n 131072
ulimit -u 8192
Reboot your system once…
reboot 

3. Create a Dedicated user for Sonarqube
The latest version of Sonar cannot run under the root user, thus create a new user to access  Sonarqube installation.

Add user
sudo adduser --system --no-create-home --group --disabled-login sonar

4. Install PostgreSQL Database
Ubuntu’s base repository doesn’t have the latest version of PostgreSQL thus to get the latest one, we have to add its repo manually. Here is the command to do that.

Add GPG key:

wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O- | sudo apt-key add -
Add repo: 

echo "deb [arch=amd64] http://apt.postgresql.org/pub/repos/apt/ focal-pgdg main" | sudo tee /etc/apt/sources.list.d/postgresql.list
Run system update

sudo apt update
Install PostgreSQL 13

sudo apt install postgresql-13
You can check the status of  its service using 

sudo systemctl status postgresql

V. Create a database for Sonar

1. Once the installation is completed, let’s create a PostgreSQL database for Sonarqube but before that set password:

sudo passwd postgres
2. Switch to postgres the user. Use the password you have set above.

su - postgres
3. Now, create a  new user that will access the database for Sonarqube.

createuser sonaruser
Note: Change sonaruser in the above command with whatever you want to use.

4. Switch to the PostgreSQL shell.

psql
5. To secure a newly created user, set a password for the same using the below syntax:

ALTER USER sonaruser WITH ENCRYPTED password 'yourpassword';
Note: change the bold items with whatever you want to use.

6. Create a new database on  PostgreSQL by running:

CREATE DATABASE sonardb OWNER sonaruser;
Note: You can use the DB name as per your choice and also don’t forget to replace the user in the above command with the one you have created.

7. Exit from the psql shell:

\q
8. Get back to your system user

exit


5. Download and Setup SonarQube on Ubuntu 20.04/18.04
While writing this article the latest version of the Sonarqube was v-9.0.1 available to download. However, you can directly visit the official website to get the latest version. You can also visit the download page and copy the link to download with wget command, as we have done here:

wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.0.1.46107.zip
Extract and move to /opt directory:

sudo apt -y install unzip
sudo unzip sonarqube-*.zip -d /opt
sudo mv /opt/sonarqube-* /opt/sonarqube
Note: If you have downloaded the file using the browser then you have to first switch to the Downloads directory before running the above commands.

Set user permission: We have created a dedicated user for SonarQube, hence, give the extracted permission to that user. 

sudo chown -R sonar:sonar /opt/sonarqube

 6. Create a SonarQube Systemd service file
By default, there will be no service file for Sonarqube to start it in the background and with system boot. Hence, we have to create one manually. Here is the way:

sudo nano /etc/systemd/system/sonar.service
Copy-paste the following lines:

[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
LimitNOFILE=65536
LimitNPROC=4096
User=sonarh2s
Group=sonarh2s
Restart=on-failure

[Install]
WantedBy=multi-user.target
Note: Replace the value of User and Group with the username that you have created at the beginning of the article for Sonarqube.

Save the file- press Ctrl+X then type- Y and hit the Enter key.

Reload the daemon:

sudo systemctl daemon-reload
Then start and enable the service

sudo systemctl enable sonar 
sudo systemctl start sonar 

Now, check whether the create Sonarqueb service running or not 

sudo systemctl status sonar
check whether the create Sonarqueb service running or not

 

[optional] Alternatively, you can also use the below commands to start, stop, and check the status:

sudo -Hu sonarh2s /opt/sonarqube/bin/linux-x86-64/sonar.sh status
sudo -Hu sonarh2s /opt/sonarqube/bin/linux-x86-64/sonar.sh start
sudo -Hu sonarh2s /opt/sonarqube/bin/linux-x86-64/sonar.sh stop
To get the console output to know what is happening while starting the Sonarqube server you can use:

sudo -Hu sonarh2s /opt/sonarqube/bin/linux-x86-64/sonar.sh console
This will help in resolving some errors.



 Alternatively, you can also use the below commands to start, stop, and check the status:

sudo -Hu sonar /opt/sonarqube/bin/linux-x86-64/sonar.sh status
sudo -Hu sonar /opt/sonarqube/bin/linux-x86-64/sonar.sh start
sudo -Hu sonar /opt/sonarqube/bin/linux-x86-64/sonar.sh stop
To get the console output to know what is happening while starting the Sonarqube server you can use:

sudo -Hu sonar /opt/sonarqube/bin/linux-x86-64/son

7. Allow Sonarqube port in Ubuntu 20.04 firewall
To access the web interface of Sonarqube you have to open its default 9000 port in your Ubuntu system’s firewall:

sudo ufw allow 9000/tcp
 

8. Access the Sonarqube Web interface
Finally, open any browser that can access the IP address or domain of the server where you have installed Sonarqube. And point it to-

http://server-ip-addres:9000
or 
http://you-somain.com:9000
 

Note: Replace server-ip-addres with your server/desktop IP address or domain name.

Log in with the default admin username

Once you see the login screen, use the default Sonarqube username and password that is admin.
