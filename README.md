### Monolithic Architecture

Monolithic software is designed to be self-contained, it is like a big container, wherein all the software components of an app are assembled and tightly coupled, i.e., each component fully depends on each other.

<img width="550" alt="monolith-arch-diagram" src="https://user-images.githubusercontent.com/69306840/184834153-8c4ab583-a4d0-4095-b1ae-7ceb948258b1.png">

### Development Environment 
Development Enviornment is a set of tools and functionalities that enable a programmer to develop, test, and debug the source code of an application or a program. 

The benefits of Dev Env is that is that developers can make changes to the code in a controlled setting without impacting the end users.

<img width="550" alt="Screenshot 2022-08-16 at 17 10 03" src="https://user-images.githubusercontent.com/69306840/184935989-b8a24748-c9dd-4694-aa46-36bbb65f7b68.png">

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

<img width="374" alt="Screenshot 2022-08-16 at 12 16 02" src="https://user-images.githubusercontent.com/69306840/184867757-0e07a5b7-ceff-484a-915f-a36eb77b9e1f.png">

### Nginx as reverse proxy
- Firstly ensure you are inside your vm. 
- Once inside run the following commands:
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



