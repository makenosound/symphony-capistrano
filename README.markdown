# Deploying Symphony with Capistrano #

By using Capistrano to manage your Symphony installation, it is possible to completely manage the Symphony deployment process, checking out the latest code, pushing it to the server. This eliminates the need for file transfers, server-side activities, and in-place/in-browser code editing.

## Setup ##

Before we get going, you'll need to make sure you've done the following:

 * Install Git and Capistrano locally
 * Ensure your application code is in a remote repository (like Github)
 * Install Git on the server
 * Ensure the deployment user on your server has SSH access to your repository

There are number of guides out there to getting setup with `git` and Capistrano, so I won't bother going into that here.

## Getting started ##

First things first. Capistrano checks for a `Capfile` in the directory you're running from and uses that to read and execute tasks. Normally to add Capistrano to a project you'd run `capify .` in your project directory, but as it's setup to work out with Rails conventions, setting up that way creates some unnecessary separation in your deployment script. For Symphony deployment we can just drop the `Capfile` from this repository into your project root on your local machine. The `Capfile` is written in ruby and contains the tasks we'll use to deploy your site.

As we're using an automated script for deployment, we make some assumptions about the setup of your project. This is pretty much in line with the basic setup that Capistrano uses, but with a few Symphony-specific tweaks. If you're serving multiple sites on a shared server, or can't modify your VirtualHosts in Apache I'd set up a `sites/` directory to deploy your project into so that you end up with something resembling the directory structure below:

    ~/
      /sites
        /example.com

Next you'll need to update the setting in the `Capfile`. You'll need to set the following:

* `:application` -- the URL of your site, is used for establishing the ssh connection and (by convention) as the folder name for deploying into
* `:deploy_to` -- the folder we're deploying into, note that we can use the `application` variable set above, or set an absolute path
* `:repository` -- the git-clone URL of your repository
* `:branch` -- the branch you want to check out, default is master
* `:user` -- the user account on your server

Once you have those things set up, just run the following command...

    cap deploy:setup
    
...from your local machine and Capistrano should log into your server and build up the folder structure ready for checking out your code. There are a number of other options you can setup (just read through the `Capfile`), but you should be able to use the defaults. If everything ran smoothly you should have something like the following on your server:

    ~/
      /sites
        /example.com
          /releases  
          /shared  
            /log  
            /manifest  
               /cache  
               /dump  
               /logs  
               /tmp  
               config.php  
            /media
            /pids  
            /system

What we have here is the basic Capistrano setup. An application folder (`~/example.com/`) with a `releases/` folder that will contain the timestamped releases of our application code and a `shared/` folder that contains files that are shared between releases. I keep the `manifest/` (configuration) files in here as well as a `media/` folder for user-uploaded content -- the `shared/media/` folder will be symlinked into the latest release once we get to that stage.

If you're using a different setup for your media/uploads then you'll need to modify the recipe. I should also mention here that we're going to be symlinking the entire `shared/manifest/` folder from the root project folder once we deploy, so you'll need to exclude that entire directory from your `git` repository (along with your `workspace/media/`). Obviously you'll want to keep some of your configuration in `git`, so I usually create a `config/` folder in my repository with a `config.example.php` file as well as any other config files that we'll need (such as JIT or CacheLite config files).

Also, the way I setup my projects is to have the root Symphony folder at the root of my `git` repository so if you have your Symphony directory in a subfolder of your repository then you'll have to change the recipe.

## Deployment ##

OK. If you're still with me we can get onto deployment. Assuming everything above went fine, all we need to do is...

    cap deploy

...and you should see Capistrano go to work logging into your server and checking out your code. Again, assuming everything went smoothly you should now see something along the lines of the below on your server:

~/
  /sites
    /example.com
      /current
      /releases  
        /20100916050732  
          Your Symphony files!
      /shared  
        /log  
        /manifest  
           /cache  
           /dump  
           /logs  
           /tmp  
           config.php  
        /media
        /pids  
        /system

Note that we now have a `current/` directory. This is a symlink to the latest release in the `releases/` folder, so you can use it as the root public folder for your site. If you're comfortable in Apache-config-land then you should be able to setup your VirtualHost file to point to this symlink, otherwise you can just create another symlink in your public folder that points to it. For example, if your server is setup to serve from `~/var/www/` then you could replace the `www/` folder with a symlink to `~/sites/example.com/current` to serve your newly deployed application instead.

## Configuration ##

In our first step (`cap deploy:setup` for the short of memory), the recipe creates an empty `config.php` file in the `shared/manifest/` folder, so you'll need to update this with your real configuration for anything to work.

## Media ##

Capistrano is pretty flexible, so you can create extra tasks (with various namespaces) for anything really. An example of this is the task I use for moving media/uploads from my local machine to/from the server. If you have a look at the `Capfile`, you'll see a couple of tasks namespaced under `:media`. These do an `scp` upload or download from the `workspace/media/` directory on your local machine to/from the `shared/media/` directory in your application folder on the server. To run them, just type:

    cap deploy:media:upload

or

    cap deploy:media:download

You'll want to be careful you don't override any files when doing this.

## Extras ##

While Capistrano is usually associated with deployment, there are a whole host of tasks it can be used to automate. The [Capistrano FAQ](http://www.capify.org/index.php/Frequently_Asked_Questions) is a good place to get started if you want to learn a little more.

I usually end up writing a few extra custom tasks for each site that do various boring things. For example, when I occasionally have to use *shudder* WordPress, I have a task that:

* dumps the local database to an .sql file
* finds-and-replaces the local URL in that dump as WordPress for some insane reason sets the URL config in the database (twice!).
* uploads the database dump to the server
* backs up the production database
* imports the new dump

I'm sure there are a whole bunch of things you could use it for -- I'm seeing Nick Dunn putting together a task that automatically dumps the latest changes from the DB Sync extension in about five minutes -- so definitely share your tasks and perhaps we can compile a more comprehensive recipe that covers most everything you'd want to do with Symphony.