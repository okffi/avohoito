
This is a constantly updated documentation for OKF Finland's main server stuff.

Language of choice is English, for fast documentation Finnish is OK if the other option would be no documentation at all.


* * *


# 0. Basic Info
* This document: [http://okf.fi/server](http://okf.fi/server)
* Contact email: [sysadmin@okf.fi](mailto:sysadmin@okf.fi) or #sysadmin channel in Slack
* Slack [https://okffi.slack.com/messages/sysadmin](https://okffi.slack.com/messages/sysadmin)
* Trello board for tickets [https://trello.com/b/N6WFJ19k/okfi-sysadmin](https://trello.com/b/N6WFJ19k/okfi-sysadmin)

Whenever you need to contact sysadmin for a certain purpose, you can
* reach out in slack #sysadmin channel (especially if you need to discuss something)
* send a task for sysadmins to do by adding it as a card to Trello
* send email

## Kapsi servers

OKFFI uses Kapsi as its main server hosting service. DNS is partly in Kapsi, partly in Louhi.


### okffi-prod1.kapsi.fi

Main production server.

8 vcpu
8 GB memory
17GB system disk
400GB data disk
ip: 91.232.156.221, 2001:67c:1be8::221

ECDSA key fingerprint is SHA256:kjFYIb3ICOWBBJg8Z8bryWB9vsM0Gu2sJ1C6UtV2xkw.

### okffi-dev1.kapsi.fi

Testing and development server.

4 vcpu
4 GB memory
21GB system disk
200 GB data disk
ip: 91.232.156.222, 2001:67c:1be8::222

ECDSA key fingerprint is SHA256:DUCWr4Uwms82/Z+H3wh9sRuL8qAM8N7cXKmOrkbbe08.

# 1. Domain names

Domains are managed mainly through Louhi (domain yearly fees). DNS services for domains may be
in Louhi or Kapsi.

Acquisition of new domains:

1. Check that the domain name is free and that the name does not appear in any trademark or company registers. [https://domain.fi/info/index/tietoa/useinkysytytkysymykset.html#312-NjNhOWYwZWE3YmI5ODA1MDc5NmI2NDllODU0ODE4NDU$61$-NXhyWmV6UTY5-0-aeFa6lBb2-aeFaSHQXc](More information for .fi domains.) If you're happy with a subdomain, in the form of example.okf.fi, that can be done more easily and without cost.

2. Contact [sysadmin@okf.fi](mailto:sysadmin@okf.fi) with the request, listing the domains you need, and which project will pay for the costs. Finnish .fi domains cost 50€ for 5 years, international domains cost $15/year (prices may change). Also let us know where the domain's website should be. We can host Wordpress easily, other sites with some setup effort, and we can point to existing outside servers as well.

# Instructions for adding new services

New services may be set up by members of OKFFI. Email the following info to sysadmin@okf.fi:
 - Service name
 - Are you putting a service up for debug or production in this phase.
 - OKFFI responsible project or working group.
 - Responsible technical person
 - Source code repository
 - Memory and data storage requirement expectations
 - Server applications required
 - Service address, SOMETHING.okf.fi, or own domain. OKFFI can also register domains, ask through slack or sysadmin(a)okf.fi
 - Repository for service configuration files. We back up also service configurations to Github. There is a private repo available, if needed.
 - Ports reserved
 - Service url for monitoring

Order a shell account http://okf.fi/sysadmin

Install the services.

Inform progress and issues through Slack

# 2.a. Installed Services and Applications (okffi-prod1)

For each service there will be service-specific system account (if needed) and someone responsible for that account. System accounts will not have passwords but will be accessed through sudo.

Components are brought to production by installing from public sources with public installation instructions. If any of these do not exist, service-specific Github repos are built before transferring services to production. It is advisable to develop and test new services with configuration management and batch deployment in mind.

## Disk space

The root file system is limited in size, so any large installations should be done in the /data file system.

## Basic services (installed)

### Security

#### Firewall

iptables

firewall with ufw

#### Apparmor

Apparmor is used in enforce mode for apache2 and other processes. Do note this when adding new web services, since new capabilities will be initially blocked. Use "aa-complain apache2" to temporarily disable enforcement, then operate the new service sufficiently, then use "aa-logprof" to approve new capabilities, and finally "aa-enforce apache" to restablish apparmor protection.

#### Acct

acct accounting logs all user logins, issued commands, and resource usage. In case of tracking down who caused a problem, this can be useful. Some key commands to try (as root):

* ac:           summarize login accounting
* last:         show the people who have logged in, and when they’ve logged out
* lastcomm:     show which commands have been used
* sa:           summarizes process accounting

NB. acct is not sufficient to prevent malicious use of granted shell privileges. It’s merely helpful in seeing who may have accidentally caused a problem in another service, while working on another one.

### Cron
Scheduled commands can be used by all users of the server. "man crontab" gives more instructions.
Essentially: by executing "crontab -e" you can edit your scheduled commands. A new line with the following contents:
`15 3 * * * cd /var/www/okf/data/FOO; git pull`
would run the "git pull" command in folder /var/www/okf/FOO every day, at 15 past 3 in the morning.

### Web server: apache2

Apache2 is the main web server.

Folders served by apache2 are mainly in /var/www. Some large installations are in /data/www and others may run from /home folders.

New virtual hosts are set up in folder /etc/apache2/sites-available, by copying an existing configuration and modifying appropriately. A new site called "NAME.conf" is taken into use like this:

0. Use existing confs as templates when creating your own. Especially if you're making a new wordpress blog into our multisite environment, copy one of the existing ones and change the server name and log files.

1. `a2ensite NAME` enables the new site configuration in Apache2.

2. `apache2ctl -t` checks that the configuration syntax is correct. **Remember to do this, since an invalid configuration file can shutdown the entire web server!**

3. `systemctl reload apache2` loads the new configuration.

If something goes wrong, do something like `a2dissite NAME` and `systemctl reload apache2` to get back to the preceding situation, so that the possible errors in your site configuration are out of the way.

4. If you need https connections, run `certbot` and choose your new site from the list. After certbot has done its thing, reload Apache.

### https certificates

EFF's Let's Encrypt is used. Https certificates are therefore free to use.
Use command `certbot` as superuser to add a certificate to any new configured domain.
A cron job runs every week to automatically renew all certificates.

### Blog service, Wordpress network/multisite

Wordpress multisite has been installed into /var/www/okf/blog.
You may request a blog to administer. It can be given a subdomain or domain controlled by OKFFI, or it can be assigned a domain owned by you.
Themes and plugins are installed by superadmins. Contact sysadmin@okf.fi if you need these.

#### Administering the blogs

Administration interface can be found at https://okffi-prod1.kapsi.fi/wp-admin/. You should have an account. Contact sysadmin if you need one.

Creation of new blogs happens like so:

1. Ask sysadmin to activate the domain or subdomain you want. These take 3-5 work days to complete, so be early.
2. Sysadmin will create a new Wordpress site for you and add you as the initial admin.
3. The site will run initially at address okffi-prod1.kapsi.fi/something
4. When the domain or subdomain DNS is set up, sysadmin can move the site to its final address. This happens by adding the domain to Apache configuration (see above), then setting up the domain mapping at http://okffi-prod1.kapsi.fi/wp-admin/network/settings.php?page=dm_domains_admin, and then going to https://okffi-prod1.kapsi.fi/wp-admin/network/sites.php and changing the "Site Address (URL)" field to the new domain. If https is enabled, use https in the URL.
5. If the site has been extensively developed in its temporary domain, it may be necessary to run Velvet Blues tool in that site's dashboard to update old URLs to new ones.

### Web analytics

We use Mamoto (formerly Piwik) for http analytics on some of our sites. Analytics needs to be enabled separately for each site where analytics are needed.. The analytics run in https://www.okf.fi/analytics/

To get an account, contact sysadmins or an existing Mamoto superuser.

## Applications

### IRC

Instructions on using screen and irssi together: [http://quadpoint.org/articles/irssi/](http://quadpoint.org/articles/irssi/)

Use the irssi client inside a screen to keep the connections alive even when you log out.

Working IRC-nodes:
* open.ircnet.org
* irc.freenode.net

Using IRC with irssi client:
#.  ssh [KÄYTTÄJÄ]@okf.fi
#.  screen -S irssi irssi
#.  Commands to run inside irssi:
* /SERVER ADD -auto -network IRCnet open.ircnet.net
* /SERVER ADD -auto -network freenode irc.freenode.net
* /CHANNEL ADD -auto #OKFFI freenode
* /CHANNEL ADD -auto #avoindata freenode
* /CHANNEL ADD -auto #OKFN freenode
#. detach from screen: ctrl-a + ctrl-d (or, in a shorter notation: ^a^d)
#. Next time, reattach the screen: `screen -r` (or `screen -x`)

You can access slack through IRC: https://okffi.slack.com/account/gateways

### URL shortener: Lessn More

URL shorterner. Access it at [http://okf.fi/-](http://okf.fi/-)

User name and password are given to people who need to create okf.fi short urls.

### Pass

Selected passwords are stored in a command-line encrypted password manager.
To access the passwords, you need sudo rights on okffi-prod1, and you need to have the GPG passphrase.
This passphrase is only given to sysadmins and core employees. If others need these passwords, they are
given upon deliberation by these individuals. Contact sysadmins for any passwords you may need.

Quick instructions:
1. Login to okffi-prod1
2. `sudo -i` to become root
3. `pass ls` to see a list of what is stored
4. `pass show [something]` to see a password; you will be prompted for the GPG passphrase

More info: https://www.passwordstore.org/


### Discourse

Discourse is installed to serve at address discussion.digirights.info.
Apache2 site has been configured to reverse proxy the Docker container that runs on port 3001 on the same server.
Docker installation is at /data/discourse

### Data web server - http://data.okf.fi (4/2018: NOT ENABLED)

/var/www/okf/data on kansio, johon kaikki käyttäjät voivat lisätä julkaistavia tiedostoja. Kansion sisältö näkyy osoitteessa http://data.okf.fi

### Paatokset.fi (4/2018: NOT ENABLED)

[http://jkl.paatokset.fi/](http://jkl.paatokset.fi/)

### GIS Geocode API (4/2018: NOT ENABLED)

Responsible admin: Mikael Rekola

Geographical Information System Geocode API

http://api.okf.fi/gis/1/autocomplete.json

http://api.okf.fi/gis/1/geocode.json

* Maanmittauslaitoksen maastotietokannan teiden geometriat ja attribuutit

* Itellan osoitteet, postinumerot, postitoimipaikat ja kunnat

* Pistemuotoiset osoitteet

[http://api.okf.fi/console/](http://data.okf.fi/console/)

[http://api.okf.fi/gis/1/geocode.json?address=Siltasaarenkatu+15](http://data.okf.fi/gis/1/geocode.json?address=Siltasaarenkatu+15)

[http://api.okf.fi/gis/1/geocode.json?lat=60.187058&lng=24.961563](http://data.okf.fi/gis/1/geocode.json?lat=60.187058&lng=24.961563)

Technically, the service is done using perl, and is located in /var/www/data/gis/

nginx proxies to apache, which uses mod_perl to call the service.

### Rahankeräysrekisteri (12/2018: NOT ENABLED)

1. asennettu Apache CouchDB 1.5.0 ja supervisor + tarvittavat lisäpaketit
2. asennettu rahankeräysrekisteri-sovellus hakemistoon /home/warmaster/WhipAroundRegistryEnvironment
3. supervisor asetettu käynnistämään WarApp-sovelluksesta 2 instanssia 127.0.0.1 porteissa 8880 ja 8881 nobody-käyttäjänä
4. supervisor kuuntelee http://127.0.0.1:9999 ja tunnukset löytyy conffista. Tuolta voi hallita WarApp-sovellusta ja seurata lokia.
5. muokattu Apache2 lataamaan proxy_balancer moduuli
6. lisätty Apache2 virtual_host konfiguraatio domaineille rahankeraysrekisteri.fi ja rahankeräysrekisteri.fi

### CKAN / Datacatalog with datastore (12/2018: NOT ENABLED)

All documentation, discussion and issues in [https://github.com/okffi/katalogi](https://github.com/okffi/katalogi)
[http://ckan.okf.fi](http://ckan.okf.fi)
Running as a docker container. See instructions at [http://docs.ckan.org/en/latest/maintaining/installing/install-using-docker.html](http://docs.ckan.org/en/latest/maintaining/installing/install-using-docker.html)

The containers have been created as follows:

$ `docker run -d --name db ckan/postgresql`

$ `docker run -d --name solr ckan/solr

$ `docker run -d -p 8081:80 --link db:db --link solr:solr --name ckan ckan/ckan`

Nginx is configured to proxy the port 8081 and serve that as virtual host ckan.okf.fi.

If the containers are not running, they can be started with

$ `docker start solr db ckan`

### Tietopyynto.fi

Python Django application based on Froide.

Customisations and settings files live in `/home/tietopyynto/tietopyynto-theme`.
Runs over mod_wsgi and Apache, using Apache config file at
`/etc/apache2/sites-available/013_tietopyynto_ssl.conf`.

Python virtualenv lives in `/home/tietopyynto/tietopyynto-env-new`

Celery is started up on system boot by `/etc/init.d/celeryd`. It calls the
script `/home/tietopyynto/tietopyynto-theme/run_celery.sh`. You can shut down
the running Celery process by killing the process number stored in
`/home/tietopyynto/tietopyynto-theme/celeryd.pid`. Give it some time to wind
down.

Logs for various related services are collected in
`/home/tietopyynto/tietopyynto-theme/tietopyynto_fi/logs/`.

Uses PostgreSQL database `tietopyyntov2`.

Uses Dovecot IMAP mailbox (localhost:143) with user `tietopyynto-mail`.

You can run Django shell or management commands with  `manage.py` by
setting the following environment variables:

```
$ export DJANGO_SETTINGS_MODULE=tietopyynto_fi.custom_settings
$ export DJANGO_CONFIGURATION=TietopyyntoProd
```

#### Troubleshooting tips

The typical problem this service has is with receiving mail, which has
by far the most moving parts in the system. Here's a checklist on
which parts can go bad.

- Check that Celery is running and capable of accepting tasks. You can
  monitor the celery.log file in the abovementioned logs directory and
  look for the once-a-minute fetch_mail task. Beat process should be
  waking up to dispatch the task every minute and MainProcess should
  subsequently acknowledge and accept it.
  - Make sure `settings.CELERY_TASK_ALWAYS_EAGER` is `False` (`from django.conf import settings` in Django shell)
- Check that the IMAP mailbox is working and accepting new mail. Try sending
  mail to it and see if you get bounces. You can peek inside the mailbox
  with a mail app, or with telnet and raw IMAP interaction. Get the username
  and password from the config file:
  - `$ telnet localhost 143`
  - `a login tietopyynto-mail <pw>`
  - `b select inbox`
  - `c search on "26-jun-2020"` <use current day to check for new mail>
  - `d fetch <latest result number from above> "(BODY.PEEK[HEADER])"` It's important to use .PEEK, otherwise the mail is marked as read and Froide won't read it again...
- Check Celery's queue broker status:
  - `# rabbitmqctl list_queues name messages consumers`
  - It should have no pending queued messages and one consumer each

* * *

# 2.b. Installed services and application (okffi-dev1)

None documented at the moment.

# 3. Emails

okf.fi email aliases are available for people who need them.

## Admin instructions
Email alias administration is done by ssh to lakka.kapsi.fi with okffi account, and editing one of these files:

* ~/.aliases/okf.fi
* ~/.aliases/mydata.org
