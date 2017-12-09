# MageConf’17 Magento Cloud workshop

## Preparation
### Add your SSH key
Sign in into [https://magento.cloud](https://magento.cloud), go to **Account settings** tab and add your public SSH key.

![Add SSH key](/images/account_settings.png?raw=true)

If you do not have SSH keys, you can generate a new key pair using the following command:
```
ssh-keygen
```
The new public key will be saved in the ~/.ssh/id_rsa/pub file.


### Clone the project
Go to the **Projects** tab and select a project. Clone the project using a git link in a web UI:

![Clone the project](/images/clone_project.png?raw=true)

```
git clone <projectid>@git.eu-3.magento.cloud:<projectid>.git mageconf2017
cd mageconf2017
```

Rest of commands will be performed from the project directory.


## Task #1: change admin password
A Magento Cloud instance as well as a deployment process can easily be customized through configuration files and environment variables.
The environment variables can be added through web UI or cli utility.

The **ADMIN_PASSWORD** variable controls a password of the admin user. To change the admin's password,
go to the **Configure environment** > **Variables** and add the following environment variable:
```
ADMIN_PASSWORD = mageconf2017
```
![Set admin password](/images/admin_password.png?raw=true)

When you save changes, a deployment process will be started in order to apply the changes.
When it is done, you can sign in to admin backend using password you have set.
You can access the admin backend using a link from web UI + **/admin**

![Access site](/images/access_site.png?raw=true)


## Task #2: clone branches
It is easy to clone branches using web UI. The codebase as well as database are copied to a new
virtual machine, then a deployment process is started in order to change BASE_URL.

To clone a branch click on the **branch** button in the top right and set a name of the new branch:

![Create branch](/images/create_branch.png?raw=true)

The current branch will be cloned and new deployment started. It will take up to few minutes to finish the clone process.

![Create branch progress](/images/create_branch_progress.png?raw=true)


## Task #3: install a custom theme
To install a custom theme just put it into app/design directory, commit and push changes.

The Dark theme is the copy of Magento/Luma theme with dark gray background.

```
mkdir -p app/design/frontend/Vendor/
curl -L https://github.com/yyevgenii/mageconf17/raw/master/resources/dark.zip -o dark.zip
unzip dark.zip -d app/design/frontend/Vendor/
rm dark.zip
git add app/design/frontend/Vendor/
git commit -m "Add Dark theme"
git push
```

If you do not have installed *curl* utility, you can try to download file with *wget*:
```
wget https://github.com/yyevgenii/mageconf17/raw/master/resources/dark.zip
```
or download the file using a web browser.


After deployment you have to switch to the Dark theme:
1. Log in to admin backend
2. Go to **Content > Configuration** page
3. Open global scope to edit
4. Switch **Applied Theme** to Magento Dark
5. Save configuration

![Switch theme](/images/switch_theme.png?raw?=true)

Go to storefront to check changes.


## Task #4: enable Elasticsearch service

Add the following lines at the end of **.magento/services.yaml**:

```
elasticsearch:
    type: elasticsearch:2.4
    disk: 1024
```


Add the elasticseach line into **relationships** section in the **.magento.app.yaml**:
```
    elasticsearch: "elasticsearch:elasticsearch"
```

The result relationships section should look like this:
```
relationships:
    database: "mysql:mysql"
    redis: "redis:redis"
    elasticsearch: "elasticsearch:elasticsearch"
```

Commit and push changes:
```
git commit -am "Enable Elasticsearch service"
git push
```

During the deployment process the Elasticsearch service will be installed and configured as well as the Magento catalog search engine will be switched to the Elasticsearch.
Go to admin backend to verify changes, the search engine settings are located at **Stores > Configuration > Catalog > Catalog > Catalog Search**:

![Catalog search](/images/catalog_search.png?raw?=true)


## Task #5: move static assets generation to the build stage

This trick decreases maintenance time by moving the **setup:static-content:deploy** step to the stage build.
On the build stage a database connection is not available and we do not know for which locales we have to generate
static assets. But we can store locale settings in a file and pass locales to the **setup:static-content:deploy** command.


For dumping the locale settings login into SSH console and run the following command:
```
php vendor/bin/m2-ece-scd-dump
```

![Dump locale settings](/images/scd_dump.png?raw?=true)

The locale settings will be dumped into a **app/etc/config.php** file. Download this file using **scp** utility, commit it and push changes to git:
```
scp z5w7ezbx255oq-scd-c4txcga@ssh.eu-3.magento.cloud:app/etc/config.php app/etc/config.php
git commit -m "Add locale settings into config.php" app/etc/config.php
git push
```

During the deployment process you will see that the static assets generation runs before maintenance mode is being enabled. In this way we have decreased a downtime caused by a deployment process.

