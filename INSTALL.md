# Getting Started Developing in RAMS: MAGFest Edition
## Getting to Know RAMS: What We Use
 If you just want to spin up an instance, feel free to skip down to "Before You Start: Setting Up Your Environment." However, if you're brand new, you should familiarize yourself with our structure.
 
### RAMS Technology
 
 RAMS is based on **Sideboard**, a custom framework that ties plugins together and runs them as a web app. The "uber" plugin - [which is its own repo](https://github.com/magfest/ubersystem) - is itself a plugin for Sideboard. In general you shouldn't need to worry about Sideboard, but be aware that it has several overrides for built-in Python libraries. It will be automatically included when you install RAMS following the instructions below.
 
 RAMS follows a MVC structure, with **CherryPy** tying the view and controller together. Every template (in `/uber/templates/`, excluding the `static_views` folder) requires a function of the same name in `/uber/site_sections`.
 
 The entire model is declared in `/uber/models.py` using the **SQLAlchemy** ORM. We use **PostgreSQL** for our database, and **Sqitch** to create and run database migrations.
 
 Currently, all configuration for the app is defined in four files: 
 * `/uber/configspec.ini` declares the static variables and includes in-line explanations of each.
 * `/development-defaults.ini` sets up sane defaults for an example event. 
 * `/development.ini`, which is not checked into Git, can be used to override any config option in your local environment. 
 * `/uber/config.py` processes these variables and inserts them into a global `c` object. Config.py also defines dynamic variables and properties, which are also added to the `c` object.
 
### MAGFest Configuration

 In order to manage the static configuration for each event, we use **Puppet** combined with **Hiera**. This allows us to define a hierarchy of .yaml files so that events and specific servers can inherit config options. However, currently each variable used in the app must be defined in the puppet manifest, `config.pp`, as well as the Ruby template, `uber-development.ini.erb`. These files are in our Puppet repo, so please be aware of them if you are changing a config option that is not already defined in our production config.
 
 The basic hierarchy, from top to bottom, is:
 * `/common.yaml`
 * `/development.yaml` and `/production.yaml`, which are used for all our staging and production servers.
 * `/event-*.yaml`, which control specific events. A -development or -production suffix means that it only applies to the staging or production server for that event.
 * `/external/*.yaml`, which are specific servers.

## Windows Instructions
### Before You Start: Setting Up Your Environment
 1. Install [GitHub For Windows](https://windows.github.com/). This will install Git for you.
 2. Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads). This is required for Vagrant.
 3. Install [Vagrant](http://www.vagrantup.com/downloads.html). This is what we currently use to deploy a consistent development environment.
 4. Set up a shortcut to launch a DOS prompt *in administrator mode.*

### Spinning Up An Event
Each event corresponds to a single instance of `/ubersystem-deploy`. If you plan on running multiple events, it is recommended you put each copy in its own folder (e.g., `/MAGFest 2016/ubersystem-deploy` and `/MAGClassic 2015/ubersystem-deploy`) 

 1. Browse to the directory you want to put your folder structure in.
 2. Using your preferred tool, clone [the deploy repo](https://github.com/magfest/ubersystem-deploy/) into that directory. The DOS command for this is `git clone https://github.com/magfest/ubersystem-deploy/`.
 3. Open up a DOS prompt in administrator mode.
 4. `cd` into the `/ubersystem-deploy` directory. You should see a Vagrantfile in this directory.
 5. Run the following commands:
```bash
vagrant up
# Vagrant will attempt to spin up a virtual server. Once it is done...
vagrant ssh
# You are now inside the Vagrant instance, which is a bash command line
cd ~/uber/puppet
# This is a one-line command to add the MAGFest production config repo to Fabric, which will pull configuration from our production config repo
{ cat fabric_settings.example.ini; echo -en "\ngit_regular_nodes_repo = 'https://github.com/magfest/production-config'";  } > fabric_settings.ini
# You can optionally use "classic" or "magstock" here instead of "prime"
./setup_vagrant_control_server.sh prime
```
 6. If all goes well, Fabric will clone down the repos you need and set up configuration to match the MAGFest event you specified. This can take up to 40 minutes.
    * During this process, you will likely be asked for your GitHub name and password. However, you actually need to use a [Personal Access Token](https://help.github.com/articles/creating-an-access-token-for-command-line-use/). Your password will not work, so don't try it. 
 7. Log out of your SSH session (`exit`) and log back in (`vagrant ssh`).
 8. Browse to `http://localhost:8000/uber/accounts/insert_test_admin` to create a test admin account.
 9. Log in with "magfest@example.com" and password "magfest".
 10. Done!

## Developing In Vagrant
There are several helpful commands that you may need while developing for RAMS.

  * In most cases, whenever you save a change to a file, RAMS will automatically restart. This will show as a 502 error in your browser while the server restarts. However, in some cases (such as a synatx error), the server does not start back up successfully. To easily see the error involved, type:
```bash
run_server
```
  * Occasionally, you will need to restart the server manually. The command for this is:
```bash
sudo supervisorctl restart all
```
  * If you add a column to a database table, simply restarting the server will be sufficient. However, if you make a different schema change, you'll need to reset your database with:
```bash
sep reset_uber_db
```
  * If you are changing MAGFest configuration, you can re-run puppet by running the following:
```bash
# This command will overwrite your development.ini file - make sure you've saved any config options you need, such as Stripe keys
~/uber/puppet/apply_node.sh
```

For more tips, see [our old development docs](https://github.com/magfest/ubersystem-deploy/blob/master/DEVELOPING.md).
