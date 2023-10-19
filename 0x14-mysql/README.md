Fresh Reset
And
Install MySQL 5.7


Note: during this guide, don't go to the next step if you have errors in the current step you are in, make sure there are no errors.

Some steps can be skipped if they don't apply to your case:

Very rare but following this guide, you may find some errors. You can try to copy-paste errors to AI WEBSITES, to help you debug them… 
	https://chat.openai.com/
	https://poe.com/
https://you.com/
https://www.phind.com/

Follow the guide, if you need clarity check this video : 
	https://youtu.be/if0DBq9OqtE

1- Clean Running MySQL Processes



Check for any running MySQL processes:
	sudo ps aux | grep mysql
If MySQL is running, try stopping it:
	sudo service mysql stop
Double-check if MySQL is no longer running:
	sudo ps aux | grep mysql
If MySQL processes are still running, terminate them:
	sudo kill -9 <PID>
		Example :  sudo kill -9 9072

Note: if you have this grep --color something, you can ignore it and continue.


Proceed to the next step when MySQL is not running.



2-Remove Existing MySQL Versions

Remove MySQL packages
	
sudo apt-get remove --purge mysql-server mysql-client mysql-common -y && sudo apt-get autoremove -y



If no errors occur, proceed to the next step.

If something like this shows up, Choose 'yes' and press Enter. 
(if it didn't show up it ok, continue)


After it finishes, it should show something like this: 


If no errors occur, proceed to the next step.




3-Remove MySQL Apt Configuration

	sudo rm -rf /etc/apt/sources.list.d/mysql.list*
	sudo rm -rf /var/lib/mysql-apt-config
	sudo dpkg --purge mysql-apt-config

Double-check that everything related to MySQL is removed:
	dpkg -l | grep mysql

It should be empty as in the image below, if not try redoing the previous steps.





4-Remove MySQL Configuration Files


sudo rm -rf /etc/mysql /var/lib/mysql



5-Edit sources.list to Remove MySQL Repositories

Check the sources.list file for MySQL repository entries (example: deb http://repo.mysql.com/apt/ubuntu bionic main), 
there should be none like the picture below:
		cat /etc/apt/sources.list | grep mysql


If there are entries, open the sources.list file:
	sudo vi /etc/apt/sources.list 

Look for (example: deb http://repo.mysql.com/apt/ubuntu bionic main) and delete lines referencing MySQL repositories. 


If no errors occur, proceed to the next step.




6-Update Packages
sudo apt update


If you got an error like this (if no error skip to step7) : 
	

	Do this command and kill all running processes (ignore grep color) : 
		ps aux | grep apt
		In this case, we have to kill this running process using their PID : 
	
	You can kill them by sudo kill -9 <PID>
	So in our case gonna be : 
		sudo kill -9 87243 
		sudo kill -9 87243 
		sudo kill -9 87497 
		sudo kill -9 87631
	Again do ps aux | grep apt : 
		Now it is good, we can skip that grep color process
	Do again sudo apt update
	
	Perfect, continue to step 7.



7-Clean APT Cache
sudo apt clean




8-Configure Any Pending Packages
sudo dpkg --configure -a





9-Install MySQL 5.7

Let's install MySQL 5.7 now (the next command is one line command) : 


sudo wget -O mysql57 https://raw.githubusercontent.com/nuuxcode/alx-system_engineering-devops/master/scripts/mysql57 && sudo chmod +x mysql57 &&  sudo ./mysql57


Note: I updated the script to select Bionic and 5.7 by default, so if you find them already selected, you can just press enter and OK… if they are not selected by default, follow with the pictures below.


A few windows are going to show up. Follow the prompts to select Ubuntu Bionic, change to MySQL 5.7, and set a password if needed.

Select Ubuntu bionic and press enter:



See the picture below, notice it shows MySQL 8.0, we will need to change it to 5.7, press enter: (if it shows 5.7 you can scroll down to OK and press enter)



After you press enter, a list of versions will show up, scroll up select 5.7, and press enter:
(skip this if already have 5.7 selected) 



After you press enter you will see something like this, just scroll down to ok and press enter, but notice that version did change to 5.7 : 



The script will continue then another window gonna show up asking you about MySQL password, feel free to enter any password, (for me to avoid problems I just leave it empty which means no password ) then press enter: 




The script will continue installing

In the end will show something like this : 



Congratulations, MySQL 5.7 is now installed correctly!





After installation, check MySQL status:
	sudo service mysql status



If issues persist, use the following commands to debug: 

	journalctl -u mysql.service

	cat /var/log/mysql/error.log

	journalctl -xe
 Creating a user and Granting Priviledges in mysql
$ mysql -root -p
Password:	/* Type root password

mysql> CREATE USER 'holberton_user'@'localhost' IDENTIFIED BY 'projectcorrection280hbtn';

mysql> GRANT GRANT REPLICATION CLIENT ON *.* TO 'holberton_user'@'localhost';

mysql> FLUSH PRIVILEGES;
Creating Database, Tables and adding Data to the Tables
mysql> CREATE DATABASE db_name_;

-- To verify if db is created
mysql> SHOW DATABASES;

mysql> USE db_name;

mysql> CREATE TABLE table_name (
    -> col_1 data_type,
    -> col_2 data_type);
-- continue adding more coloums to your taste for me i just added two coloumns

mysql> INSERT INTO table_name VALUES ("val_1", "val_2");

-- Verify if data was added succesfully do
mysql> SELECT col_1, col_2 FROM tb_name;
Setting Up MySQL Replication
First create replication user and grant replication priviledge ( best practice).
mysql> CREATE USER 'replica_user'@'%' IDENTIFIED BY 'replica_user_pwd';

mysql> GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'%';

mysql> FLUSH PRIVILEGES;

-- to verify
mysql> SELECT user, Repl_slave_priv FROM mysql.user;

mysql> exit
Next up you go to the /etc/mysql/mysql.conf.d/mysqld.cnf and comment the bind address and then add this lines to it
# By default we only accept connections from localhost
# bind-address = 127.0.0.1
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = db_name
Then you enable incoming connection to port 3306 and restart mysql-server
$ sudo ufw allow from 'replica_server_ip' to any port 3306

$ sudo service mysql restart
Now log back in to mysql-server to lock db and prepare binary file for replication.
$ mysql -uroot -p
password:
mysql> 

mysql> FLUSH TABLES WITH READ LOCK;

mysql> SHOW MASTER STATUS;

+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      149 | db           |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
Take note of the binary log and the position, jot it down or you leave this window open and you open another window to continue

you then export the db from myql-server to local machine and then copy this db to replica machine
$ mysqldump -uroot -p db_name > export_db_name.sql

$ scp -i _idenetity_file_ export_db_name.sql user@machine_ip:location
Then ssh to replica machine ip_adress to import this tables to replica mysql-server
$ mysql -uroot -p 
password:


mysql> CREATE DATABASE db_name;

mysql>exit
bye

$ mysql -uroot p db_name < export_db_name.sql
password:

# Now edit the config file in /etc/mysql/mysql.conf.d/mysqld.cnf and then reload mysql-server

```bash

server-id = 2
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = db_name_from_master_mysql-server
relay_log = /var/log/mysql/mysql-relay-bin.log

$ sudo service mysql restart
Login to mysql server in replica to configure replication
$ mysql -uroot -p
password:


mysql>
mysql> CHANGE MASTER TO
    -> MASTER_HOST='source_host_name',
    -> MASTER_USER='replication_user_name',
    -> MASTER_PASSWORD='replication_password',
    -> MASTER_LOG_FILE='recorded_log_file_name',
    -> MASTER_LOG_POS=recorded_log_position;

-- Then you start slave
mysql> START SLAVE;
