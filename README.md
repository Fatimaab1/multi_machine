### Monolithic Architecture

Monolithic software is designed to be self-contained, it is like a big container, wherein all the software components of an app are assembled and tightly coupled, i.e., each component fully depends on each other.

<img width="550" alt="monolith-arch-diagram" src="https://user-images.githubusercontent.com/69306840/184834153-8c4ab583-a4d0-4095-b1ae-7ceb948258b1.png">

### Development Environment 
Development Enviornment is a set of tools and functionalities that enable a programmer to develop, test, and debug the source code of an application or a program. 

The benefits of Dev Env is that is that developers can make changes to the code in a controlled setting without impacting the end users.

<img width="600" alt="Screenshot 2022-08-18 at 09 59 55" src="https://user-images.githubusercontent.com/69306840/185355752-a8bb6f0f-a28e-4596-b15d-3f93d6312a7b.png">


### Multi Machine

Creating two virtual machines called App and DB

- Add the following code to vagrantfile:
```
Vagrant.configure("2") do |config|

    config.vm.define "app" do |app| # this created a virtual machine called app
        app.vm.box = "ubuntu/bionic64" # sets up linux 
        app.vm.network "private_network", ip: "192.168.56.10" # nginx web server for app virtual machine network setup 
        app.vm.synced_folder ".", "/home/vagrant/app" # this syncs data from LH
        app.vm.provision :shell, path: "provision.sh" # shell provisioner for script and provision.sh location path 

    end



    config.vm.define "db" do |db|
        db.vm.box = "ubuntu/bionic64"
        db.vm.network "private_network", ip: "192.168.56.11"

    end

end
    
```

You then need to follow the commands below:

- `vagrant destory`
- `vagrant up`
(ERROR: Install pm2 not working, added privision.sh code manually to find error. Solved: remove pm2)
- `vagrant shh app` - when accessing a specific machine you need to specify which one you want to ssh in
- `cd app`
- `cd app`
- `npm install`
- `npm start`

Then exit, you now need to go into DB machine using the following commands:
- `vagrnant ssh db`
- `sudo apt-get update -y`
- `sudo apt-get upgrade -y`


## Env Var

### Linux variable & Env variable in Linux - Windows - Mac

- How to check existing Env Var `env` or `printenv`
- How to create a var in Linux `Name=Fatima`
- How to check Linux Var `echo $Name`
- Env var we have a key word caled `export` command is `export Last_Name=Barkat`
- Check specific Env var `printenv Last_name` - outcome should be 'Barkat' 

##### How to make Env variable `PERSISTENT`
DB_HOST=mongodb://192.168.56.151:27017/posts

##### steps:
- edit the .bashrc file:
- Write a line for each variable you wish to add using the following syntax: `export Name='', export Last_Name='', export DB_HOST=''` and ctrl x then y to save.
- Next enter the following command `source .bashrc`
- Result

<img width="600" alt="Screenshot 2022-08-18 at 09 59 55" src="https://user-images.githubusercontent.com/69306840/185355473-83065f95-1afd-4a6f-99b5-9e5e8c11e19d.png">


### Nginx as reverse proxy
- Run the following commands:
- `sudo nano /etc/nginx/sites-available/default`- 
- The file will then open for editing
- Within the server block you will find `location/` block, in here you need to replace the content inide with `proxy_pass http://localhost:8080;` 
- Now save changes made using ctrl x and then y
- To ensure syntax is correct run `sudo nginx -t` 
- If there were no errors after running the above command you need to run sudo `systemctl restart nginx` to restart nginx
- Then run npm start and you should see this message 'Your app is ready and listening on port 3000'
- Go to your browser and your app should be running 

### Reverse Proxy 

- Create a new file and name it using `sudo nano reverse_proxy`
- Inisde the text editor write the following code:
```
server {
        listen 80;
        listen [::]:80;

        access_log /var/log/nginx/reverse-access.log;
        error_log /var/log/nginx/reverse-error.log;

        location / {
                    proxy_pass http://localhost:3000;
  }
}
```
- You then need to add to your provision.sh file using the following commands:
```
sudo cp -f app/app/proxy_file /etc/nginx/sites-available/default
sudo systemctl restart nginx

```
- Once provision file is saved exit the current location
- Inside your local machine run `vagrant destory`
- `vagrant up`
- You now need to go back into your app virtual machine using `vagrant ssh app`
- run `cd app` x2
- Then run `npm install`
- `npm start`
- You should see the followiing message 'Your app is ready and listening on port 3000'
- Now check refresh your browser (add http:// if neccessary)


### Automating Mongodb
- Create a new provision file for db (provisiondb.sh) and add the following code:
```
# update
sudo apt-get update -y


# upgrade
sudo apt-get upgrade -y


sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927
echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20

systemctl status mongod
sudo systemctl restart mongod
sudo systemctl enable mongod


sudo cp -f app/app/mongodb.conf /etc/mongod.conf
sudo systemctl restart mongod
sudo systemctl enable mongod
sudo systemctl status mongod
```
- In the existing provision.ssh add the following code:
```

echo "DB_HOST=mongodb://192.168.56.11:27017/posts" | sudo tee -a /etc/environment
printenv DB_HOST

#seed
cd app/app
npm install
cd seeds
node seed.js
```
- Then create a new file mongo config file 'mongodb.conf' and add the following:
```
# mongodb.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0

```

- Then follow the commands below:
- `vagrant destroy`
- `rm -rf .vagrant`
- `vagrant up`
