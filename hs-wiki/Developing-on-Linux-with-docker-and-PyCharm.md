This document is part of the [Hydroshare Developers' Guide](https://github.com/hydroshare/hydroshare2/wiki/Hydroshare-Developers'-Guide)

# 1. Prerequisites

* [PyCharm](http://www.jetbrains.com/pycharm/download/)
* [Docker](https://www.docker.io/)
* Git 
* (optional) a graphical Git client which supports submodules such as [Sourcetree]( http://www.sourcetreeapp.com/)

# 2. Setup

This document will get you setup with a Hydroshare development environment.  The configuration of this environment exactly reflects the configuration on `beta.hydroshare.org` and `dev.hydroshare.org` to ensure that code that works in one place works in all places. The only exception to this is IRODS, which is not run on the developer machines.

## 2.2 Setup fig

Fig is a wrapper around docker that makes it easier to run an application.  We recommend you familiarize yourself with all the fig commands.  The documentation is short and to the point.  [It cam be found here](http://www.fig.sh/cli.html)

    $ sudo curl -L https://github.com/docker/fig/releases/download/0.5.2/linux > /usr/local/bin/fig
    $ sudo chmod +x /usr/local/bin/fig

## 2.3 Clone git repository

All the information that Hydroshare needs to build itself is in the GitHub repository.  If you are not familiar with GitHub, it has extensive online help.

    $ git clone https://github.com/hydroshare/hydroshare2
    $ cd hydroshare2

## 2.4 Update submodules

Hydroshare uses submodules to track extensions and other repositories it depends on.  Currently the submodules are `hs_core` and `django_irods`, however other modules such as user profile information may be added.  Git submodules are a matter of some controversy and their usage can be complicated.  Please read up on the issues regarding submodules in your spare time. Among other common issues, you can get submodules into a "detached head" state. **This can cause you to lose work if you aren't careful!**

    $ git submodule init
    $ git submodule update

## 2.5 Build base hydroshare container

This is a Docker container that encapsulates all dependencies of Hydroshare in terms of the Linux environment and the Python environment.  It is separate from the main Hydroshare Dockerfile because it can take up to an hour to build and this makes it so that the build step only has to happen once.  In `hs_docker_base` there is a `requirements.txt` file and a `Dockerfile`.  The `Dockerfile` contains the build instructions.  The requirements.txt contains all required Python packages.  If you add a Python package you must update `requirements.txt` and rebuild by following this step.  If you add a system package (through apt-get or by downloading and building a package) you must edit `Dockerfile` and rebuild by following this step.

    $ cd hs_docker_base
    $ docker build .

    ...
    Successfully built 07c3abde3387

    $ docker tag 07c3abde3387 hs_base

For the last command, you **must** "tag" the image built by the Dockerfile `hs_base`.  This step registers the dependency in Docker's catalogue.  Without it, your `fig build` statement in the next section will fail with a 404 error.  To tag the image, copy the hexadecimal image ID from the line that starts with `Succesfully built` and paste it into the terminal window as the first argument of `docker tag` as seen above.

## 2.6 Build Hydroshare

    $ cd ..
    $ fig up > hydroshare.log 2> hydroshare.err &

Note that because of database configurations, it's likely that **you will get an error the first time** you type `fig up`.  This is expected and okay.  Simply type `fig start` after it errors out and it will run fine.

## 2.7 Restore development database

In the hydroshare2 repository are two files for boostrapping your database and hydroshare environment.  They create a new environment.  **The root account for the created environment is** username **root**, password **root**.

    $ ./restore-db hydroshare-development-db.sql
    $ ./restore-media-linux hydroshare-development-media.sql

**Note** that you can copy the file restore-media to restore-media-linux if restore-media-linux does not exist, and replace the hardcoded ip address with localhost in the new restore-media-linux file.  

## 2.8 Check your work

You should now be able to go to [http://localhost:8000](http://localhost:8000) and see a Hydroshare welcome screen. If you do not see the welcome screen but instead see a "Page not found" yellow page from django, try to go to [http://localhost:8000/admin](http://localhost:8000/admin) and login with root (password: root), then click on "Sites" from the left side menu, and change the site record to "localhost:8000" from "192.168.59.103:8000"

# 3 Setup PyCharm

### 3.1.1 Configure Source Control

In the File menu, click `Open directory` and select the hydroshare repository you just cloned in the section **Get a local copy of hydroshare2 from GitHub**.  In the example pictured in img. 8 it is `hydroshare_pycharm`, but for you it is just where-ever you cloned hydroshare2.

_Image 8:_

![open directory](https://raw.github.com/hydroshare/hydroshare2/master/docs/pycharm8.png)

### 3.1.2. Configure Git project roots

At this point, PyCharm knows where your code is, but it doesn't know about GitHub.  To correct that, you will want to register your Git roots with PyCharm.  Open the "Settings" menu.  It may be called "Preferences" on a Mac.  If the screen that comes up doesn't look like the img. 9 image below when you open it, click the "Version Control" item from the menu on the left hand and click "add root" on all the red boxes. This process is pictured in img. 10.

_Image 9:_

![configure roots](https://raw.github.com/hydroshare/hydroshare2/master/docs/pycharm9.png)

_Image 10:_

![configure roots](https://raw.github.com/hydroshare/hydroshare2/master/docs/pycharm10.png)

### 3.1.3 Configure Deployment

This section helps you configure PyCharm so that the code you develop in the IDE gets deployed to the container running on your virtual machine when you want to test it.  All the steps in this subsection (steps **a** through **e**) require that you are in the "Settings" (or "Preferences" on some systems) dialog. 

#### (a.) Open the Settings dialog. Pick the item _Project > Deployment > Options_, and follow the instructions below:

* Check “Create empty directories”
* Check “Delete target items when source ones do not exist”
* Switch “Upload changed files automatically to the default server” to “Always”  (pictured in img. 11)
* Click "Apply"

_Image 11:_

![general settings](https://raw.github.com/hydroshare/hydroshare2/master/docs/pycharm11.png)

#### (b.) Add a server for Deployment:

* Pick the item _Project > Deployment_.
* Click the `+` sign to add a new Deployment target.  The dialog pictures in img. 12 below will show up.  
* Give it a name. 
* Pick the SFTP protocol if it is not already picked.
* Click OK. 

_Image 12:_

![server for deployment](https://raw.github.com/hydroshare/hydroshare2/master/docs/pycharm12.png)

#### (c.) Edit the connection settings:
 
* Click the `Connection` tab in the dialog.
* Your settings should look like this:
    * **SFTP Host**: `localhost`
    * **Port**: `1338`
    * **Root Path**: `/`
    * **Username**: `docker`
    * **Auth type**: `password`
    * **Password**: `docker`
    * **Web server root url**: `http://localhost:8000`
* Click "Test SFTP Connection" to make sure that settings work.  If they do not, send a support request to hydroshare-dev
* Click "Apply"

#### (d.) Setup the deployment mapping:

* Pick the item _Project > Deployment_.
* Click the `Mappings` tab in the dialog.
* Set the local path to the path were you cloned Hydroshare2
* Set the deployment path on the server to `/home/docker/hydroshare`
* Set the web path on the server to `/`
* Click "Apply"

The results should look like this:

#### (e.) Enable Django support. 

* Pick the item _Django Support_
* Enable Django support by checking on the box.

### 3.2.4 Configure remote interpreter

This section helps you configure PyCharm so that you can run and test code in an interpreter and so that PyCharm can syntax check and perform autocompletion using the code and libraries in the container.  All the steps in this subsection require that you are in the Settings (or Preferences on some systems) dialog. 

* Pick the item _Project Interpreter_
* Click the `+` item to add a new interpreter (see img. 15)
* Configure it as a "remote interpreter" (see img. 16)
* Click "fill from deployment server settings" (see img. 17)
* Click the name of the deployment server you setup
* Click OK (see img. 18)

_Image 15:_

![project interpreter](https://raw.github.com/hydroshare/hydroshare2/master/docs/pycharm15.png)

_Image 16:_

![project interpreter](https://raw.github.com/hydroshare/hydroshare2/master/docs/pycharm16.png)

_Image 17:_

![project interpreter](https://raw.github.com/hydroshare/hydroshare2/master/docs/pycharm17.png)

_Image 18:_

![project interpreter](https://raw.github.com/hydroshare/hydroshare2/master/docs/pycharm18.png)

# 4. Common tasks

## 4.2.1. View server logs

    $ less hydroshare.log
    $ less hydroshare.err

## 4.2.2. Stop Hydroshare

    $ fig stop

## 4.2.3. Restart Hydroshare

When it has been built once already:

    $ fig start

If your machine becomes corrupt or unresponsive, or you just need to rebuild to get to a good state:

    $ fig kill
    $ fig rm
    $ fig build
    $ fig up > hydroshare.log 2> hydroshare.err &
    $ ./restore-db-linux <latest good restore>
    
Then right click the hydroshare project in your IDE and click on Deployment.../Upload.  Finally: 

    $ fig run --rm hydroshare python manage.py migrate

## 4.2.4. Restart boot2docker

    $ boot2docker restart
 

## 4.2.5. Change database schema to reflect changes in a models.py file

Sync-migrations will simplify using migrations in this workflow. It will will create migrations on the server side, apply them to your database, and then copy them back into your Hydroshare project tree at the appropriate place. You should run the following script anytime you edit models.py to add or remove a database field:

    ~/hydroshare $ ./sync-migrations (app-name)

Where `app-name` is the name of the Hydroshare sub-project (hs_*) whose models.py has changed.  When you run this script, the migrations will get copied back to your app's migrations directory and you will need to add them in git yourself.

Also, when you pull the latest versions of hs_core and hydroshare2, please run sync-migrations, as the resource type has been updated to support URL shortening, and all AbstractResource subclasses will have a new field added to them via migration.  

## 4.2.6. Customizing Hydroshare with an Extension

Please see the "Working on your own Hydroshare extension" section of the [Customizing Hydroshare](https://github.com/hydroshare/hydroshare2/wiki/Customizing-Hydroshare#wiki-working-on-your-own-hydroshare-extension) document for instructions on how to add a module.

## 4.2.7. Backup and restore databases

From within your hydroshare base directory, backup and restore are as follows:

    $ ./backup-db-linux
    $ ./backup-media-linux
    $ ./restore-db-linux <name of backup sql file>
    $ ./restore-media-linux <name of backup media targz>

## 4.2.8. Run a Python shell

    $ fig run --rm hydroshare python manage.py shell

## 4.2.9. Run management commands
    
    $ fig run --rm python manage.py <command name> <args...>

# 6. Troubleshooting and References

## Errors

**If you find yourself accidentally in a detached HEAD state,** then use the Tools menu in PyCharm to "Open a Terminal" and type this:

    $ git checkout -b fixit1 
    $ git checkout master
    $ git pull origin master
    $ git merge fixit1
    $ git branch -d fixit1

Essentially, by entering these commands, you are saving your changes make while in a detached HEAD state to a temporary branch ("fixit1"), merging them with the master branch, and then deleting your temporary branch. You will need to complete this process in all branches that are in this state.

You can read more about detached heads in [this git-notes article](https://github.com/sitaramc/git-notes/blob/master/articles/detached-head.mkd).

## Errors on server about missing tables or fields, or other problems with the Docker container

If your machine becomes corrupt or unresponsive, or you just need to rebuild to get to a good state:

    $ fig kill
    $ fig rm
    $ fig build
    $ fig up > hydroshare.log 2> hydroshare.err &
    $ ./restore-db-linux <latest good restore>
    
Then right click the hydroshare project in your IDE and click on Deployment.../Upload.  Finally: 

    $ fig run --rm hydroshare python manage.py migrate