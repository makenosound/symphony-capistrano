#------ Capistrano recipe for deploying Symphony CMS ---------------------
#  Max Wheeler - http://makenosound.com/

# Pull in default Capistrano recipes
load 'deploy'

#------ Settings ------------------------------

# The name of your website
set :application, "tasks.makenosound.com"

# The path to your deployment directory on the server
# by default, the name of the application (e.g. "/home/user/projects/example.com")
set :deploy_to, "/home/maxw/#{application}"

# The git-clone url for your repository
set :repository, "ssh://git@icelab.repositoryhosting.com/icelab/icelab-site.git"
set :branch, "master"

# The name of the deployment user-account on the server
set :user, "maxw"


#------ Default setup -------------------------

#  Standard setup below, won't need editing if unless you're customising

#  SCM settings
set :scm, :git
set :ssh_options, { :forward_agent => true } # Uses your public key for authentication
set :deploy_via, :remote_cache
set :copy_strategy, :checkout
set :use_sudo, false
set :copy_compression, :bz

# Number of previous releases to keep
set :keep_releases, 10

# Automatically checkout submodules?
set :git_enable_submodules, 1 

# Server roles
role :app, "#{application}"
role :web, "#{application}"
role :db,  "#{application}", :primary => true


#------ Deployment order ---------------------

after "deploy:setup", "deploy:create_directories" 
after "deploy:update", "deploy:cleanup" 
after "deploy", "deploy:set_permissions", "deploy:create_symlinks"

#------ Deployment tasks ---------------------
namespace :deploy do
  desc "This overides the default :restart"
  task :restart, :roles => :app do
    # Do nothing but overide the default
    # If you need to restart and processes, add them here
  end

  task :finalize_update, :roles => :app do
    run "chmod -R g+w #{latest_release}" if fetch(:group_writable, true)
    # Overide the rest of the default method
  end

  desc "Create additional Symphony directories and set permissions after initial setup"
  task :create_directories, :roles => :app do
    # Create env. specific config directories
    run "mkdir #{deploy_to}/#{shared_dir}/manifest"
    run "mkdir #{deploy_to}/#{shared_dir}/manifest/tmp"
    run "mkdir #{deploy_to}/#{shared_dir}/manifest/cache"
    run "mkdir #{deploy_to}/#{shared_dir}/manifest/logs"
    run "mkdir #{deploy_to}/#{shared_dir}/manifest/dump"
    # Create an empty config.php for setup purposes
    run "touch #{deploy_to}/#{shared_dir}/manifest/config.php"
    run "mkdir #{deploy_to}/#{shared_dir}/media"
    # Set permissions
    run "chmod 755 #{deploy_to}/#{shared_dir}/manifest"
    run "chmod 775 #{deploy_to}/#{shared_dir}/manifest/tmp"
    run "chmod 775 #{deploy_to}/#{shared_dir}/manifest/cache"
    run "chmod 775 #{deploy_to}/#{shared_dir}/manifest/logs"
    run "chmod 755 #{deploy_to}/#{shared_dir}/manifest/config.php"
    run "chmod 775 #{deploy_to}/#{shared_dir}/media"
  end

  desc "Set the correct permissions for the workspace/media files"
  task :set_permissions, :roles => :app do
    # Set permissions for everything
    run "chmod -R 755 #{current_release}"
    # Set writable permissions for workspace files
    run "if [ -e #{current_release}/workspace/data-sources ]; then chmod -R 775 #{current_release}/workspace/data-sources"
    run "if [ -e #{current_release}/workspace/events ]; then chmod -R 775 #{current_release}/workspace/events"
    run "if [ -e #{current_release}/workspace/pages ]; then chmod -R 775 #{current_release}/workspace/pages"
    run "if [ -e #{current_release}/workspace/utilities ]; then chmod -R 775 #{current_release}/utilities"
    # Set writable permissions for shared media directory
    run "chmod 775 #{deploy_to}/#{shared_dir}/media"
  end

  desc "Create symlinks to shared data"
  task :create_symlinks, :roles => :app do
    run "ln -s #{deploy_to}/#{shared_dir}/manifest #{current_release}/manifest"
    run "ln -s #{deploy_to}/#{shared_dir}/media #{current_release}/workspace/media"
  end

  desc "Clear the Symphony caches, including /logs and /tmp files"
  task :clear_cache, :roles => :app do
    run "if [ -e #{current_release}/manifest/cache ]; then rm -r #{current_release}/manifest/cache/*; fi"
    run "if [ -e #{current_release}/manifest/logs ]; then rm -r #{current_release}/manifest/logs/*; fi"
    run "if [ -e #{current_release}/manifest/tmp ]; then rm -r #{current_release}/manifest/tmp/*; fi"
  end
  
  #------ Rollback tasks [deploy:rollback] ---------------------
  
  namespace :rollback do
    desc "Points the current symlink at the previous revision. This is called by the rollback sequence, and should rarely (if ever) need to be called directly."
    task :revision, :except => { :no_release => true } do
      if previous_release
        run "rm #{current_path}; ln -s #{previous_release} #{current_path}"
      else
        abort "Could not rollback the code because there is no previous release"
      end
    end
    
    desc "Removes the most recently deployed release. This is called by the rollback sequence, and should rarely (if ever) need to be called directly."
    task :cleanup, :except => { :no_release => true } do
      run "if [ `readlink #{current_path}` != #{current_release} ]; then rm -rf #{current_release}; fi"
    end
    
    desc "Rolls back to the previously deployed version. The `current' symlink will be updated to point at the previously deployed version, and then the current release will be removed from the servers."
    task :code, :except => { :no_release => true } do
      revision
      cleanup
    end
    
    desc "Rolls back to a previous version and restarts. This is handy if you ever discover that you've deployed a lemon; `cap deploy:rollback' and you're right back where you were, on the previously deployed version."
    task :default do
      revision
      cleanup
    end
  end
  
  #------ Media tasks [deploy:media:task] ---------------------
  
  namespace :media do
    desc "Copy local /workspace/media contents to the shared media folder on the server"
    task :upload do
      top.upload("workspace/media/*", "#{deploy_to}/#{shared_dir}/media", :via => :scp, :recursive => true)
    end
    
    desc "Copy contents of the shared media folder on the server to the local /workspace/media directory"
    task :download do
      top.download("#{deploy_to}/#{shared_dir}/media/*", "workspace/media", :via => :scp, :recursive => true)
    end
  end
end