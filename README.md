# Deploy Django in AWS EC2

## 1. Setting up server

* Allow HTTP (Port 80), SSH (Port 22), HTTPS (Port 443) inbound traffic of your server.
* Save your server connection files in a secure location and connect through the SSH port of your server.
* Windows users follow [this guide](https://asf.alaska.edu/how-to/data-recipes/connect-to-your-ec2-instance-using-putty-v1-1/) to connect using the **Putty** client.
* Once logged in run these commands to update the server ubuntu os in to the latest stable version.
```bash
sudo apt-get update
```
* Install following packages to the server.
```bash
sudo apt install python3-pip python3-dev python3-venv libpq-dev postgresql postgresql-contrib nginx gunicorn curl
```
* Then enable firewall for nginx and openssh(Optional).
```bash
sudo ufw enable
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
```

## 2. Configuration for Github

* Install Github CLI 
```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh
```
* Then upgrade CLI
```bash
sudo apt update
sudo apt install gh
```
* Add your Github to server(When generating tokens add a token with a limited amount of time).
```bash
gh auth login
```
* Finally clone your repository
```bash
git clone <YOUR-PROJECT-GIT>
```

### 3. Configuring Postgresql
* Open the psql terminal.
```bash
sudo -u postgres psql
```
* Execute following commands one at a time to create a db and a user.
```bash
CREATE DATABASE dbname;
```
```bash
CREATE USER user WITH ENCRYPTED PASSWORD 'pass';
```
```bash
GRANT ALL PRIVILEGES ON DATABASE dbname TO user;
```

### 4. Configuring environment(one time only)
* Open the cloned the repository and execute following commandsto install venv.
```bash
cd <YOUR-PROJECT-NAME>
python3 -m venv env
source env/bin/activate
pip install wheel gunicorn
pip install -r requirements.txt
```
* Add the .env file content to the server environment(Keep no spaces in variable value assigning) and restart shell.
```bash
sudo nano /etc/environment
```
* Then do the migrations for django.
```bash
python manage.py makemigrations
python manage.py migrate
python manage.py collectstatic
``` 
* Create a superuser to django.
```bash
python manage.py createuser
```

## 5. Configuring Gunicorn
* Create a socket file for gunicorn.
```bash
sudo nano /etc/systemd/system/gunicorn.socket
```
* Add the content of the [gunicorn.socket](gunicorn.socket) file to the editor window.
* Next configure the gunicorn.service file.
```bash
sudo nano /etc/systemd/system/gunicorn.service
```
* Add the content of the [gunicorn.service](gunicorn.service) file to the editor window with replacing necessary content.
* Run following to make gunicorn service start and enabled.
```bash
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
sudo systemctl status gunicorn.socket
```

### 6. Configuring NGINX
* Create a seperate file for the project
```bash
sudo nano /etc/nginx/sites-available/<YOUR-PROJECT-NAME>
```
* Add the content of [nginx](nginx) file to the editor window after replacing the necessary content.
* Once your NGINX config is set up, make sure there are no syntax errors.
```bash
sudo nginx -t
```
* Next, create a soft link of your config file from sites-available to the sites-enabled directory.
```bash
sudo ln -s /etc/nginx/sites-available/<YOUR-PROJECT-NAME> /etc/nginx/sites-enabled
```
* Run following to make gunicorn service start and enabled.
```bash
sudo systemctl restart nginx
```

### 7. Automate Deployment
* Copy the deployment file(if available) to the root directory with other required files.
```bash
cp <YOUR-PROJECT-NAME>/deploy.sh .
``` 
* Then excute deployment.
```bash
/bin/sh deploy.sh
``` 
