# Deploying Symphony with Capistrano #

By using Capistrano to manage your Symphony installation, it is possible to completely manage the Symphony deployment process, checking out the latest code, pushing it to the server. This eliminates the need for file transfers, server-side activities, and in-place/in-browser code editing.

While Capistrano is usually associated with deployment, there are a whole host of tasks it can be used to automate. The [Capistrano FAQ](http://www.capify.org/index.php/Frequently_Asked_Questions) is a good place to get started if you want to learn a little more.

## Requirements ##

Before we get started, you'll need to make sure you've done the following:

 * Install Git and Capistrano locally
 * Ensure your application code is in a remote repository (like Github)
 * Install Git on the server
 * Ensure the deployment user on your server has SSH access to your repository

## Setup ##

As we're creating an automated script for deployment, the script make some assumptions about the setup of your project.

If you're on a shared server, or can't modify your VirtualHosts in Apache

## Directory structure ##

    /sites
      /example.com
        /current (set Apache DocumentRoot to this directory)  
        /releases  
        /shared  
          /manifest  
             /cache  
             /dump  
             /logs  
             /tmp  
             config.php  
          /media

Note: the media file is a shared media folder symlinked from `workspace/media/`


## .gitignore ##

manifest
workspace/media

