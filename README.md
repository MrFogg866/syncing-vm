# Syncing vagrant to Virtual Machine and running a app

![alt](img/vagrant.png)

## Create a Vagrant file

1. Make sure to be in the correct folder in vs code by using `cd` iv put mine in the virtualisation folder.
2. Then we use the command `touch vagrant` to make a vagrant file in the repo.
3. We then change the ‘base’ to look like this.

 `Vagrant.configure("2") do |config|`

  `config.vm.box = "ubuntu/xenial64”`

![alt](img/vagrant-config.png)



4. We then use `vagrant up` to access vagrant from vs code


## Create a provision file for script


1. We `touch provision.sh` to create a provision shell file
2. we `nano` into the file 
3. Then add this script to the file in vscode 

`#!/bin/bash`

`sudo apt-get update -y` 
`sudo apt-get upgrade -y`
`sudo apt-get install nginx -y`




![alt](img/provision.png)

4. we then have to change permissions using `chmod`



3. Then we add this line into the vagrant file as the 3rd line 

`config.vm.provision "shell", path: "provision.sh”`

4. The we run `vagrant up` to check the script is running and you should see everything being executed

## syncing the app folder 

1. After adding the app folder to the repo
2. In the vagrant file add `config.vm.synced_folder "app", "/home/vagrant/app”`
3. Ensure its in the correct place by using `pwd` in vscode it should show /home/vagrant from the terminal
4. The do `vagrant up` 
5. Then in bash - `vagrant ssh`then use `ls` to check the app folder has synced over to the virtual machine

## install ruby to run tests and check 

1. `cd` into environment folder and then into spec-tests
2. `sudo gem install bundler` - without sudo may give permission failure
3. Should see “1 gem installed”
4. Then type `bundle` this will bundle the tests up
5. Then `rake spec` which runs all tests, this checks what is present and what isn’t
6. Should be 9 checks installed if not, install what isn’t installed.
7. We then go into vagrant on bash by using the `vagrant ssh` command
8. Then install `sudo apt-get install nodejs -y`
9. Then install `sudo apt-get install python-software-properties`
10. Use this command to get a specific version `curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -` of node js 
11. Then run `sudo apt-get install nodejs -y` again to grab the version in the above step
12. To check the version can run `nodes —version
13. Install node package manager into the app folder `sudo npm install pm2 -g` then `npm install` 
14. And `npm start`
15. Should see a message saying “Your app is ready and listening on port 3000”
16. To check enter the ip in the browser followed by :3000 and you should see this
![alt](img/welcome.png)



# npm vs pm2 

- nmp = node package manager
- pm2 = nodes process manager



# nginx reverse proxy 

1. What are ports? (research)
- In computing, a port or port number is a number assigned to a uniquely identify a connection endpoint, and to direct data to a specific service.


2. What is a reverse proxy? 

![alt](img/reverse-proxy.png)

- this is a layer of protection that sits in front of web servers and forwards client (e.g. web browser) requests to those web servers. A reverse proxy protects web servers from attacks and can provide performance and reliability benefits. ip address hidden from user, also for load balancing these are used and handling SSL Encryption

3. How is it different to a proxy? 

![alt](img/forward-proxy.png)

- a forward proxy protects the clients online identity, bypasses browsing restrictions and can be used to restrict certain content, it sits between the clients and the internet to do this, a reverse proxy sits between the internet and the web servers. A simplified way to sum it up would be to say that a forward proxy sits in front of a client and ensures that no origin server ever communicates directly with that specific client.



4. What is Nginx's default configuration (hint - 'sites-available' directory)

- to check this we can run `cat /etc/nginx/nginx.conf` to check default config and we will see the default configuration and this is port 80 

![alt](img/nginx-config.png)
## How do you set up a Nginx reverse proxy?

### Setting up NGINX as reverse proxy
The following guide explains how to set up nginx as a reverse proxy:

1. Start your VM by using vagrant up in VS Code terminal. For this, ensure that app.js is not being deployed automatically in your provisioning.sh
2. Connect to your VM with Bash terminal by navigating into your project folder with cd command and then use vagrant ssh command
3. Use command cd /etc/nginx/sites-available in order to navigate inside the nginx configuration folder
4. Create a new configuration file with nano by using the following command: sudo nano nodeapp.conf (name could be anything, but try to make logical)
5. Inside the file type the following code:
6. 
```
server {
   listen 80;
   server_name 192.168.10.100;

   location / {
       proxy_pass http://192.168.10.100:3000;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
   }
}
```

`server_name ***  - can be the name of your server or your ip address`
6. Use ctrl+x to exit nano, then press y to save the changes, and then press enter to save the name of the file
7. Enable the configuration by creating a symbolic link to enable a new config file: sudo ln -s /etc/nginx/sites-available/nodeapp.conf /etc/nginx/sites-enabled/nodeapp.conf
8. Check configuration for errors sudo nginx -t
9. Reload nginx - sudo systemctl reload nginx
10. Go back to your home folder by using cd command and then navigate in to cd app
11. Launch app by using node app.js
12. Paste your ip, without port number, into browser and check if it works: 
