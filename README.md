# Linux Server Configuration

Full Stack Nanodegree Program Project #7. Tasked with setting up a Linux server running Ubuntu 16.04 LTS to host the Item Catalog web application.

## Important Details

__Public IP Address:__ 54.255.235.252
__SSH Port:__ 2200
__Public URL (IPv4):__ <http://ec2-54-255-235-252.ap-southeast-1.compute.amazonaws.com/catalog/>

## Steps Taken

1. **Setting up your Amazon Lightsail instance**
	1. Navigate to [Amazon Lightsail](https://lightsail.aws.amazon.com)
	2. Create an account or log in if you already have an account
	3. Create an instance image. This will be OS only, and on Ubuntu 16.04 LTS (LTS = Long Term Support)
	4. Choose the lowest plan when prompted to do so, it should be only $5.
	5. Give your instance a name.

2. **Update your packages**
	1. Click the big yellow button which says "Connect using SSH". A dialog box will open with you connecting to your instance.
	2. Once in, type `sudo apt-get update` to check for any available updates.
	3. Once you see the prompt again, type `sudo apt-get upgrade` to upgrade all packages that need updating.
	4. This step is optional, but I always do a `sudo apt-get autoremove` to remove all packages which are no longer depended upon.

3. **Creating a new user with sudo permissions**
	1. Change to the root user with `sudo su`
	2. Create the new user by typing `adduser grader`
	3. Give the new user - grader - sudo access by typing `vi /etc/sudoers.d/grader` and add the line `grader ALL=(ALL) NOPASSWD:ALL`
	4. Restart the shell with `service ssh restart`

4. **Creating SSH key for new user**
	1. Type in `ssh-keygen` on your local machine and specify the place where your file should be saved. A good practice is to save it in `~/.ssh`.
	2. Once generated, navigate to the folder with `cd ~/.ssh` and there will be two keys. Copy the contents of the `key.pub` file. There will be your user at the end of the line, something like `user@localMachine`. Delete those few words and ensure there is no space after the last letter once deleted.

5. **Modifying SSH port and disabling root login**
	1. Create a `.ssh` directory for grader with `sudo mkdir /home/grader/.ssh`
	2. Copy the authorized keys from the standard user, ubuntu, with `sudo cp /ubuntu/.ssh/authorized_keys /home/grader/.ssh/authorized_keys`
	3. Change the permissions for the folder and file with `chmod 700 /home/grader/.ssh/` and `chmod 644 /home/grader/.ssh/authorized_keys`
	4. Copy the contents of the previously `key.pub` file and paste them using `sudo vi /home/grader/.ssh/authorized_keys`
	5. Change the owner and group for the folder and file with `chown grader /home/grader/.ssh` and `chown grader /home/grader/.ssh/authorized_keys` and replace `chown` with `chgrp` to change the group.
	6. Navigate to the `sshd_config` file with `sudo vi /etc/ssh/sshd_config` and change `Port` to `2200`
	7. Modify `PermitRootLogin without-password` to `PermitRootLogin no`
	8. Change `PasswordAuthentication without-password` to `PasswordAuthentication no`
	9. Restart the shell with `service ssh restart`
	10. Login with the new grader user with `ssh grader@ip -p 2200 -i ~/.ssh/key`. The file to reference is the private key, which is the file without the `.pub` extension.

6. **Configuring timezone to UTC**
	1. Type `sudo dpkg-reconfigure tzdata` into the command line as grader
	2. Select "None of the above" and then choose "UTC" when you see it.

7. **Configuring Uncomplicated Firewall (UFW)**
	1. In the terminal, type `sudo ufw default deny incoming`. This is to block all incoming connections.
	2. After, type `sudo ufw default allow outgoing`. This is to allow all outgoing connections.
	3. Next, type `sudo ufw allow 2200/tcp`. This is to allow the port 2200 which will be used for SSH.
	4. Then, type `sudo ufw allow www`. This is to allow connections to the World Wide Web, which is also HTTP port 80.
	5. Type in `sudo ufw allow ntp`. This is to allow connections to NTP protocol, which is TCP port 123.
	6. Lastly, enable your firewall by typing `sudo ufw enable`
	7. Do a quick check to see the current firewall rules by typing `sudo ufw status`

8. **Clone the Item Catalog repository**
	1. Type `sudo apt-get install git` if it is not already installed. If it is, skip this step.
	2. Next, type `cd /var/www/` and then `sudo mkdir ItemCatalog`. Once done, type `cd ItemCatalog` and then type `git clone https://github.com/YOUR_USER/YOUR_REPOSITORY_NAME.git`
	3. Once done, follow the steps provided by [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) to set up your Flask app on the server.

8.1 **Debugging**
	1. Take note, there is an error with your app when you receive a 500 Internal Server Error response when you navigate to your home page. Try to debug until you get what you are supposed to see. Also, do look in the logs at `/var/log/apache2/error.log` to get a better idea of what's wrong.
	2. While following the steps listed above, there might be times when you run into trouble. If you get the error message saying that you do not have a particular module, e.g `ImportError: No module requests found`, type sudo -H pip install <NAME_OF_MODULE>. The -H parameter in sudo is to install the package for all users. 
	3. The app might have trouble finding your `client_secrets.json` file, and so simply put the absolute path into your code. For example, instead of `client_secrets.json`, type `/var/www/ItemCatalog/ItemCatalog/client_secrets.json`. 
	4. Do not forget to update your `client_secrets.json` file to enable OAuth. Since this app is only using Google Accounts, navigate to the project page, and then change the Authorized Javascript Origin to the public address, e.g. `http://ec2-54-255-235-252.ap-southeast-1.compute.amazonaws.com`. Change the redirect URL to `http://ec2-54-255-235-252.ap-southeast-1.compute.amazonaws.com/catalog/`, then replace the `client_secrets.json` file with the new file. If you can't figure out your hostname, key in your IP address at th:is website [here](http://www.hcidata.info/host2ip.cgi).
	5. If you get an SQLAlchemy related error, it's either the fact that you haven't installed the module, or because you haven't modified the parameters in `create_engine()`. I will address this later.

9. **Setting up PostgreSQL on the instance**
	1. Start off by typing `sudo apt-get install postgresql postgresql-contrib`
	2. On installation, PostgreSQL creates a user named `postgres`. Change to the user using `sudo su - postgres` and then type `psql` in the terminal.
	3. As required, create a PostgreSQL role called `catalog`. You can do this by `CREATE ROLE catalog WITH PASSWORD password;`
	4. Once created, create the catalog database with `CREATE DATABASE catalog WITH OWNER catalog`
	5. Check your permissions on all users by typing `\du` in the `psql` terminal.
	6. Once done, modify all scripts that interact with the database. The key line you are looking for is `create_engine(sqlite3:///'http://localhost:5000')`. Modify the parameters to make it look like `create_engine('postgresql://catalog:YOUR_PASSWORD@localhost/catalog')`.
	7. Once you have modified all scripts with the above change, run your database script with `python database_setup.py` and your test data with `python test_data.py`
	8. Once ready, restart Apache with `sudo service apache2 restart`

10. **Launch the App**
	1. Check that your app is running fine on your public URL, which is <http://ec2-54-255-235-252.ap-southeast-1.compute.amazonaws.com/catalog/> for me.

## Acknowledgements

[Setting up a Flask Application on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
<br>
[Using PostgreSQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)

