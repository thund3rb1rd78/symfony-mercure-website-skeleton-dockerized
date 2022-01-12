Based/forked off of Dunglas's symfony-docker project, so props to him for getting this started for me: https://github.com/dunglas/symfony-docker

This is a production ready, HTTPS enabled (across the board, including Mercure and db admin area) website skeleton, images based on: symfony website-skeleton v6.0, caddy web server v2, mercure v0.13, mariadb v Latest, and phpMyAdmin v Latest. I also added the UX Turbo, Mercure, and UX Turbo / Mercure bridge bundles to Symfony already.

I have tested this on a fresh install of centOS, and it works out of the box, YRMV. At first I wasn't able to get much going on a VPS that I had all kinds of other things install on it, i.e. apache, nginx, several phps, mysql, etc., so a fresh linux is advised.
This also works out-of-the-box locally with https, but you have to 'accept the security risk', there's different ways of doing this per browser.

Below is a list of notable changes I made to Dunglas's project.
- Removed all postgres database related lines of code. There were multiple locations of these.
- Added a line to the Dockerfile so pdo and pdo_mysql extentions are installed in the php server, this is at line 100 if you want to swap out for your flavor db.
- Removed 'auto-script' code from composer.json that cleared Symfony cache and ran assest:install. These commands, when doctrine bundle is installed, wants to connect to the database (IDK why?), this makes the docker-compose build to fail because the database is, obviously, not running. So, you'll have to remember to clear cache as needed and run assests:install command manually, if needed.
- Removed ! from Mercure keys, it borks up the docker up command when running in prod, I forget why, I think it could possibly be escaped? But it was simpler to just not use them.
- Obviously, added the mariadb image and phpmyadmin image (with their respective volumes) to docker-compose.yml
- Added a DATABASE_URL line to the environment section of the php image in docker-compose.yml
- Added some env variables in .env to help with things in docker-compose.yml
- Added lines 17-20 in docker/caddy/Caddyfile, this allows you to reach your db admin utilizing Caddy's auto-issued SSL through a little reverse proxy that sends traffic to the db_admin container. Without this you would need to expose a port from that container and use that to get to the db admin area, and it wouldn't be encrypted. Two small caveats with this, after you log in it kicks back you to your symfony homepage. Just go back to the /dbAdmin/ URL and you'll be logged in. Also, stating the obvious, you can not use any url that uses /dbAdmin/ in your Symfony app, it will not work. And second, the trailing slash is REQUIRED. Like this https://localhost/dbAdmin/ . These are small prices to pay, imo, to get simple out of the box working db admin area that is encrypted. Maybe someone will improve on it one day? But for me it's fine.

To build and start this locally run:
docker-compose build --pull --no-cache
docker-compose up

I will admit, I’m really new to docker environments. And there’s a couple of things I’ve noticed already. When working locally, the changes you make in your project folder are reflected in the docker container. There's no reason to rebuild or down/up the docker. Also, your updates that are done through 'yarn watch' for webpack/encore are also reflected real time. Something important to keep in mind though. You don't have access to the database server from your IDE or standard shell. Therefore, commands that want connection to the database must be done from the docker php container because it has access to the docker db container. Two examples of what I mean here. First, ‘symfony console make:controller’ will work from either desktop IDE/shell OR inside container shell, because it doesn’t need the db connection. However, ‘symfony console make:entity’ must be done from the container shell.

To build and start on production
docker-compose build --pull --no-cache
SERVER_NAME=subdomain.yourDomain.com APP_SECRET=ChangeMe CADDY_MERCURE_JWT_SECRET=ChangeMe docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
