<div id="top"></div>

<!-- PROJECT SHIELDS -->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]



<!-- PROJECT LOGO -->
<br />
<div align="center">

<h3 align="center">Key price comparer</h3>

  <p align="center">
    A simple website that compares the prices of steam keys from different sellers  <br/>
    <a href="https://youtu.be/PxzwyMUr13A">View Demo</a>
    ·
    <a href="https://github.com/kais-btl/keydeals/issues">Report Bug</a>
 </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->

## About

### Built With

* [Python 3](https://www.python.org)
* [Flask](https://flask.palletsprojects.com/en/2.1.x/)
* [Nginx](https://www.nginx.com)
* [uwsgi](https://uwsgi-docs.readthedocs.io/en/latest/#)
* [Mysql (mariaDB)](https://mariadb.org)
* [datatables](https://datatables.net)
* [Bootstrap](https://getbootstrap.com)

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- GETTING STARTED -->

## Getting Started

These are some tips and steps on how to set this project up and how I did it myself.
### Prerequisites

- A domain name (a free .ga .ml etc. works)
  - I used a cloudflare account to manage the DNS and proxy the domain
- A linux machine to host the website
- nginx to host the site
    ```bash
    sudo apt install nginx
    ```
- mariadb to host the database
    ```bash
    sudo apt-get install mariadb-server
    sudo mysql_secure_installation
    ```
  - set up a user to access this db via the flask app and edit the credentials in app.py 
   (only local access is needed but remote access is nice to have)
### Installation

1. Clone the repo
   ```bash
   git clone https://github.com/kais-btl/keydeals.git
   ```
2. Move into the folder
3. Install python modules (preferably in a virtual environment - see python3-venv)
   ```bash
   pip install -r requirements.txt
   ```
   
#### database
1. database setup script:
	```mysql
	DROP TABLE IF EXISTS keydeals.store;
	DROP TABLE IF EXISTS keydeals.app;
	DROP TABLE IF EXISTS keydeals.catalogue;

	DROP SCHEMA IF EXISTS keydeals;
	CREATE SCHEMA keydeals;
	USE keydeals;
	SET NAMES utf8mb4;
	ALTER DATABASE keydeals CHARACTER SET = utf8mb4 COLLATE = utf8mb4_bin;

	CREATE TABLE keydeals.store (
	    storeId INT NOT NULL PRIMARY KEY,
	    name VARCHAR(50) NOT NULL,
		link VARCHAR(150) NOT NULL,
	    icon VARCHAR(50)
	);

	CREATE TABLE keydeals.app (
	    appId INT NOT NULL PRIMARY KEY,
	    name VARCHAR(100) NOT NULL
	);


	CREATE TABLE keydeals.catalogue (
	    price FLOAT NOT NULL,
	    appId INT NOT NULL,
	    StoreId INT NOT NULL,
		link VARCHAR(150) NOT NULL,
	    PRIMARY KEY (appId , storeId),
	    FOREIGN KEY (appId)
		REFERENCES keydeals.app (appId),
	    FOREIGN KEY (storeId)
		REFERENCES keydeals.store (storeId)
	);
	```

###  Hosting:
#### uwsgi

1. install and setup uwsgi (check if the site runs on localhost 5000)
    ```bash
    pip install uwsgi
    uwsgi --socket 0.0.0.0:5000 --protocol=http -w wsgi:app
    ```

3. create a systemd service to run the server
   ```bash
   sudo nano /etc/systemd/system/keydeals.service
   ```
4. populate the keydeals.service file with the following config
   ```service
    [Unit]
    Description=uWSGI instance to serve keydeals
    After=network.target
    
    [Service]
    User=<username>
    Group=www-data
    
    WorkingDirectory=/home/<username>/www/keydeals
    Environment="PATH=/home/<username>/www/keydeals/venv/bin"
    ExecStart=/home/<username>/www/keydeals/venv/bin/uwsgi --ini app.ini
    
    [Install]
    WantedBy=multi-user.target
   ```
5. start the service
   ```bash
   sudo nano /etc/systemd/system/keydeals.service
   ```
6. enable it at boot
    ```bash
    sudo systemctl enable keydeals
    ```
#### Nginx
6. edit/create the project site config
    ```bash
    sudo nano /etc/nginx/sites-available/keydeals
    ```
    ```
    server {
        root /var/www/keydeals.ga;
        index index.html index.htm index.nginx-debian.html;
    
        listen 80;
        server_name keydeals.ga www.keydeals.ga;
    
        location / {
            include uwsgi_params;
            uwsgi_pass unix:/home/mypi/www/keydeals/app.sock;
        }
   ```
7. Link the available site to enabled
    ```bash
    sudo ln -s /etc/nginx/sites-available/keydeals /etc/nginx/sites-enabled
    ```
8. Unlink default if needed
    ```bash
    sudo unlink /etc/nginx/sites-enabled/default
    ```
9. restart nginx
    ```bash
    sudo systemctl restart nginx
    ```
#### SSL (https)
10. install certbot and sign SSL certificate for your website
    ```bash
    sudo apt install certbot python3-certbot-nginx
    sudo certbot --nginx -d keydeals.ga -d www.keydeals.ga
    ```
    
#### scraping scheduler
1. set up a cron job for the scraper
    ```bash
    crontab -e
    ```
2. fill in the command and timer you want for the `schedule` command
    ```
    5 10 * * * cd /home/<username>/www/keydeals && venv/bin/flask schedule >>scheduled.log 2>&1
    ```

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- USAGE EXAMPLES -->

## Screenshots
<p align="right">(<a href="#top">back to top</a>)</p>
What you end up with is a webpage with a datatable that you can search and sort for games you want to check.

### cloudflare dashboard
![cf](https://imgur.com/UmJXkoq.png)

### table
![table](https://imgur.com/Lv0WEEg.png)

### search
![search](https://imgur.com/6P88BDT.png)



<!-- ROADMAP -->

## Roadmap

- [ ] Server sided datatable
- [ ] page with game details
- [ ] notifications for when db updates
- [ ] Nested Feature


<p align="right">(<a href="#top">back to top</a>)</p>


<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->

[contributors-shield]: https://img.shields.io/github/contributors/kais-btl/keydeals.svg?style=for-the-badge
[contributors-url]: https://github.com/kais-btl/keydeals/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/kais-btl/keydeals.svg?style=for-the-badge
[forks-url]: https://github.com/kais-btl/keydeals/network/members
[stars-shield]: https://img.shields.io/github/stars/kais-btl/keydeals.svg?style=for-the-badge
[stars-url]: https://github.com/kais-btl/keydeals/stargazers
