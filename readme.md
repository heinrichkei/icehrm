IceHrm
===========
[![Build Status](https://travis-ci.org/gamonoid/icehrm.svg?branch=master)](https://travis-ci.org/gamonoid/icehrm)

IceHrm is an [HRM software](https://icehrm.com) which enable companies to manage employee details and HR workflows.

- Checkout IceHrm without installing: [IceHrm Demo](https://icehrm.com/icehrm-demo)
- Get a Managed IceHrm hosting: [IceHrm Cloud](https://icehrm.com/icehrm-cloud)
- Self-hosted IceHrm with all the features in IceHrm Cloud: [IceHrmPro](https://icehrm.com/purchase-icehrmpro)

![](docs/images/icehrm-open-demo-1.png)
&nbsp;&nbsp;&nbsp;&nbsp;
![](docs/images/icehrm-open-demo-2.png)

## Installation

### Using AWS EC2

#### 1. Launch an EC2 Instance:

- Choose an Amazon Machine Image (AMI): Select an Ubuntu Server LTS AMI.
- Instance Type: Choose an instance type that meets your performance requirements (e.g., t2.micro for testing).
- Configure Security Group: Open the following ports:
  - HTTP (port 80): For web traffic.
  - HTTPS (port 443): For secure web traffic.
  - SSH (port 22): For secure shell access.

#### 2. Connect to Your EC2 Instance:

- Use SSH to connect to your instance:
  ```bash
  ssh -i /path/to/your-key.pem ubuntu@your-ec2-public-ip
  ```

#### 3. Update System Packages:

- Update the package list and upgrade existing packages:
  ```bash
  sudo apt update && sudo apt upgrade -y
  ```

#### 4. Install Apache & MySQL:

- Install Apache:
  ```bash
  sudo apt install apache2 -y
  ```
- Install PHP and Extensions:
  ```bash
  sudo apt install php libapache2-mod-php php-mysql php-gd php-xml php-mbstring -y
  ```
- Install MySQL Server:
  ```bash
  sudo apt install mysql-server -y
  ```

#### 5. Secure MySQL Installation:

- Run the security script to set the root password and secure MySQL:
  ```bash
  sudo mysql_secure_installation
  ```

#### 6. Create a MySQL Database and User for IceHrm:

- Access the MySQL shell:
  ```bash
  sudo mysql -u root -p
  ```
- Execute the following commands, replacing `icehrm_db`, `icehrm_user`, and `strong_password` with your desired database name, username, and password:
  ```sql
  CREATE DATABASE icehrm_db;
  CREATE USER 'icehrm_user'@'localhost' IDENTIFIED BY 'strong_password';
  GRANT ALL PRIVILEGES ON icehrm_db.* TO 'icehrm_user'@'localhost';
  FLUSH PRIVILEGES;
  EXIT;
  ```

#### 7. Download and Configure IceHrm:

- Navigate to the web root directory:
  ```bash
  cd /var/www/html
  ```
- Run the following command to check the current permissions in your `/var/www/html` directory:
  ```bash
  ls -lah /var/www/html
  ```
- If you see `root` as the owner and your current user is `ubuntu` (default for AWS), change ownership:
  ```bash
  sudo chown -R ubuntu:ubuntu /var/www/html
  ```
- Use `wget` to dynamically fetch the latest IceHrm release:
  ```bash
  LATEST_RELEASE_URL=$(curl -s https://api.github.com/repos/gamonoid/icehrm/releases/latest | grep "browser_download_url" | cut -d '"' -f 4)
  ```
- Download the Latest IceHrm Release:
  ```bash
  wget $LATEST_RELEASE_URL
  ```
- If the file is in `.zip` format, install `unzip` (if not already installed) and extract:
  ```bash
  sudo apt install unzip -y
  unzip icehrm_v*.zip
  ```
- Move the Extracted Files:
  ```bash
  sudo mv icehrm_v* icehrm
  ```
- Set the Correct Permissions:
  ```bash
  sudo chown -R www-data:www-data /var/www/html/icehrm
  sudo chmod -R 755 /var/www/html/icehrm
  ```

#### 8. Configure Apache for IceHrm:

- Create a new Apache configuration file for IceHrm:
  ```bash
  sudo nano /etc/apache2/sites-available/icehrm.conf
  ```
- Add the following configuration, replacing `your_domain` with your actual domain or public IP:
  ```bash
  <VirtualHost *:80>
    ServerAdmin admin@your_domain
    DocumentRoot /var/www/html/icehrm
    ServerName your_domain
    <Directory /var/www/html/icehrm>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
- Enable the new site and rewrite module:
  ```bash
  sudo a2ensite icehrm.conf
  sudo a2enmod rewrite
  ```
- Restart Apache to apply the changes:
  ```bash
  sudo systemctl restart apache2
  ```

#### 9. Complete the IceHrm Installation via Web Browser:

- Open your web browser and navigate to `http://your_domain` or `http://your-ec2-public-ip`.
- Follow the on-screen instructions to complete the installation:
  - Database Host: `localhost`
  - Database Name: `icehrm_db`
  - Database User: `icehrm_user`
  - Database Password: `strong_password`
- After installation, log in with the default credentials:
  - Username: `admin`
  - Password: `admin`

#### 10. Secure the Installation:

- For security reasons, delete the installation directory:
  ```bash
  sudo rm -rf /var/www/html/icehrm/app/install
  ```
- Update the admin password immediately after the first login.

### Using Docker

- Install docker on Mac, Windows or Linux [https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)
- Download the [latest version of IceHrm](https://github.com/gamonoid/icehrm/releases/latest) and extract it.
- Alternatively you can cone the repo `git clone https://github.com/gamonoid/icehrm.git`
```bash
cd icehrm
npm run setup
npm run docker:build
npm run docker:start
```

![](docs/images/IceHrm-installation.gif)

- Visit [http://localhost:3128/](http://localhost:3128/) and login using `admin` as username and password.
- Visit [http://localhost:3130/](http://localhost:3130/) to access phpmyadmin.
- All user uploaded files are stored under `icehrm/docker/production/app_data`

### Installation (without docker)
- Please check [Installation guide](https://icehrm.com/explore/docs/installation/).

### Upgrade from Previous Versions

- Download the [latest release](https://github.com/gamonoid/icehrm/releases/latest).
- Extract the release file.
- Replace the `icehrm/core` directory in your current installation.
- Replace the `icehrm/web` directory in your current installation.
- Copy or replace `icehrm/updater` in your current installation.
- From `icehrm/app` directory in release, copy and replace following files in your installtion `icehrm/app` directory:
  - fileupload-new.php
  - fileupload_page.php
  - google-connect.php
  - updater.php

## Setup Development Environment
```bash
git clone https://github.com/gamonoid/icehrm.git
cd icehrm
docker compose up -d
```
- Visit [http://localhost:9010/](http://localhost:9010/) and login using `admin` as username and password.
- Watch this for more detailed instructions: [https://www.youtube.com/watch?v=sz8OV_ON6S8](https://www.youtube.com/watch?v=sz8OV_ON6S8)

### Extend IceHrm with custom Extensions
- Inorder to create an admin extension run
```bash
php ice create:extension sample admin
```

![](docs/images/icehrm-create-ext.gif)


- Refresh IceHrm to see a new menu item called `Sample Admin`
- The extension code can br found under `icehrm/extensions/sample/admin`
- Refer: [https://icehrm.com/explore/docs/extensions/](https://icehrm.com/explore/docs/extensions/) for more details.

### Building frontend assets

- When ever you have done a change to JavaScript or CSS files in icehrm/web you need to rebuild the frontend
- First make sure you have all the dependencies (just doing this once is enough)
```bash
cd icehrm/web
npm install
cd ..
npm install
```

- Build assets during development
```bash
gulp clean
gulp
```

- Build assets for production
```bash
gulp clean
gulp --eprod
```

- Build extensions
```bash
gulp ejs --xextension_name/admin
```

### Debugging code with psysh
You can run psysh inside the icehrm web docker container to manually debug the code.
- Start Psysh console
``` bash
docker compose up -d
docker exec -it icehrm-icehrm-1 /bin/sh
./psysh -c ./.config/psysh/config.php
```
This will open a psysh console. You can instantiate any IceHrm class and debug it.
Here is an example of creating an employee object and loading an employee from the database.
```bash
$emp = new \Employees\Common\Model\Employee();
$emp->Load('id = ?',[1]);
var_dump($emp);
```

### Running tests (Docker)

- Run e2e (cypress) tests

```bash
docker compose -f docker-compose-testing.yaml up --exit-code-from cypress
```
or
```bash
docker compose -f docker-compose-testing.yaml up --exit-code-from cypress --build --force-recreate
```

- When you are ready to push your changes to production, make sure to build the production images
```bash
docker compose -f docker-compose-prod.yaml up -d --build
```

### Useful Links
* IceHrm Opensource Blog: [http://icehrm.org](http://icehrm.org)
* IceHrm Cloud Hosting: [https://icehrm.com](https://icehrm.com)
* IceHrm Documentation (Opensource and Commercial): [https://icehrm.com/explore/docs/](https://icehrm.com/explore/docs/)
* IceHrm Blog: [https://icehrm.com/blog](http://icehrm.com/blog)
* Purchase IceHrm Pro: [https://icehrm.com/modules.php](https://icehrm.com/modules.php)
* Report Issues: [https://github.com/gamonoid/icehrm/issues](https://github.com/gamonoid/icehrm/issues)
