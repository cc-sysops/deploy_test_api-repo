#!/usr/bin/env rake
require "bundler/setup"
require 'json'
require 'rake'
#require 'rspec/core/rake_task'
require 'mixlib/shellout'

## Build Packer Image
# require 'erb'
# require 'yaml'

cfg_dir = File.expand_path File.dirname(__FILE__)

desc 'Install Databag Defined Cookbooks'
task :cookbooks_install do

  require 'json'
  require 'mixlib/shellout'

  contents = JSON.parse(File.read(File.join(cfg_dir, "data_bags/test_api/cookbook_version.json")))

  unless ::File.exists?(::File.join(cfg_dir, "site-cookbooks"))
    command_mkdir = Mixlib::ShellOut.new("rm -rf #{::File.join(cfg_dir, "site-cookbooks")}", :cwd => cfg_dir)
    command_mkdir.run_command
  end

  if ::File.exists?(::File.join(cfg_dir, "site-cookbooks/test_api_app"))
    command_rm = Mixlib::ShellOut.new("rm -rf site-cookbooks/test_api_app", :cwd => cfg_dir)
    command_rm.run_command
  end
  # shell_out!("git clone git@github.com:cc-sysops/test_api_app-cookbook site-cookbooks/test_api_app")

  command_clone = Mixlib::ShellOut.new("git clone git@github.com:cc-sysops/test_api_app-cookbook site-cookbooks/test_api_app", :cwd => cfg_dir)
  command_clone.run_command

  command_tag = Mixlib::ShellOut.new("git checkout #{contents['test_api_app']['revision']}", :cwd => "#{cfg_dir}/site-cookbooks/test_api_app")
  command_tag.run_command

  first = "ENV['BERKSHELF_PATH'] = File.expand_path('../vendor', __FILE__)"
  content = Array.new
  content << first
  require 'json'
  File.open('site-cookbooks/test_api_app/Berksfile', 'r') do |f1|  
    while line = f1.gets
      #puts "WHATEVER -> #{line =~ /metadata/}"  
      content << line unless ( line =~ /metadata/ )
    end
  end

  content << 'cookbook "test_api_app", path: "site-cookbooks/test_api_app"'

  puts content

  ::File.open(::File.join(cfg_dir, 'Berksfile'), 'w') do |data_bag_item|
    data_bag_item.puts content
  end

  berks_vendor = Mixlib::ShellOut.new("berks vendor vendor/cookbooks") # -b #{File.join("site-cookbooks", "test_api_app", "Berksfile")}")#, File.join(cfg_dir, "site-cookbooks/ccbind")
  berks_vendor.run_command

end

desc 'Install cookbooks using berkshelf'
task :berks_vendor do
  require 'json'
  require 'mixlib/shellout'

  contents = JSON.parse(File.read(File.join(cfg_dir, "data_bags/test_api/cookbook_version.json")))

  # puts contents.inspect
  # puts contents['test_api']['revision']
  #  contents.each do |k,v|
  #    v.each do |y|
  berks_vendor = Mixlib::ShellOut.new("berks vendor vendor/cookbooks -b #{File.join("site-cookbooks", "test_api_app", "Berksfile")}")#, File.join(cfg_dir, "site-cookbooks/ccbind")
  berks_vendor.run_command
  #  end

  # cba = Array.new
  # Dir.glob("cookbooks/*").each do |cb|
  #   # cba << "-b #{File.join(cb, "Berksfile")}"
  #   sh "bundle exec berks vendor #{cb}-cookbooks -b #{File.join(cb, "Berksfile")}"#, File.join(cfg_dir, "site-cookbooks/ccbind")
  # end

  # # puts cba.join("/berksfile -b ")
  # # sh "bundle exec berks vendor cookbooks #{cba.join(" ")}"#, File.join(cfg_dir, "site-cookbooks/ccbind")

end

desc "Lists all the tasks."
task :list do
  puts "Tasks: \n- #{(Rake::Task.tasks).join("\n- ")}"
end

# ENV['BERKSHELF_PATH'] = cfg_dir + '/.berkshelf'

desc 'Ensure skeleton files are up to date'
task :skeleton do
  begin
    require 'git'
    g = Git.open('.')

    remotes = g.remotes.map { |r| r.name }
    rname = 'skeleton'
    unless remotes.include?(rname)
      puts 'Adding skeleton remote to your repository'
      git_uri = 'https://github.com/cloudware-cookbooks/skeleton.git'
      g.add_remote(rname, git_uri)
    end

    # fetch & merge remote
    puts 'fetching latest bones'
    g.remote(rname).fetch
    puts 'merging remote branch'
    sh "git merge -X theirs -m 'skeleton cookbook sync' --squash #{rname}/master"
  rescue => e
    warn 'The skeletons in your closet are unhappy'
    puts e.message
  end
end

#desc "Run all tests"
#task :test => []
#task :default => :test

#begin
  #require "kitchen/rake_tasks"
  #Kitchen::RakeTasks.new

  #desc "Alias for kitchen:all"
  #task :integration => "kitchen:all"

  #task :test => [:cookbooks_install, :integration]
#rescue LoadError
  #puts ">>>>> Kitchen gem not loaded, omitting tasks" unless ENV['CI']
#end


# desc "Checks for required dependencies."
# task :check do
#   environemnt_vars = [
#     'OPSCODE_USER',
#     'OPSCODE_ORGNAME',
#     'KNIFE_CLIENT_KEY_FOLDER',
#     'KNIFE_VALIDATION_KEY_FOLDER',
#     'KNIFE_CHEF_SERVER',
#     'KNIFE_COOKBOOK_COPYRIGHT',
#     'KNIFE_COOKBOOK_LICENSE',
#     'KNIFE_COOKBOOK_EMAIL',
#     'KNIFE_CACHE_PATH'
#   ]
#   errors = []
#   environemnt_vars.each do |var|
#     if ENV[var].nil?
#       errors.push(" - \e\[31m#Variable: {var} not set!\e\[0m\n")
#     else
#       puts " - \e\[32mVariable: #{var} set to \"#{ENV[var]}\"\e\[0m\n"
#     end
#   end

#   # client_key               "#{ENV['HOME']}/.chef/#{user}.pem"
#   # validation_client_name   "#{ENV['ORGNAME']}-validator"
#   # validation_key           "#{ENV['HOME']}/.chef/#{ENV['ORGNAME']}-validator.pem"
#   # chef_server_url          "https://api.opscode.com/organizations/#{ENV['ORGNAME']}"

#   file_list = [
#     "#{ENV['KNIFE_CLIENT_KEY_FOLDER']}/#{ENV['OPSCODE_USER']}.pem",
#     "#{ENV['KNIFE_VALIDATION_KEY_FOLDER']}/#{ENV['OPSCODE_ORGNAME']}-validator.pem",
#   ]

#   file_list.each do |file|
#     if File.exist?(file)
#       puts " - \e\[32mFile: \"#{file}\" found.\e\[0m\n"
#     else
#       errors.push(" - \e\[31mRequired file: \"#{file}\" not found!\e\[0m\n")
#     end
#   end

#   if system("command -v virtualbox >/dev/null 2>&1")
#     puts " - \e\[32mProgram: \"virtualbox\" found.\e\[0m\n"
#   else
#     errors.push(" - \e\[31mProgram: \"virtualbox\" not found!\e\[0m\n")
#   end

#   if system("command -v vagrant >/dev/null 2>&1")
#     puts " - \e\[32mProgram: \"vagrant\" found.\e\[0m\n"
#     %x{vagrant --version}.match(/\d+\.\d+\.\d+/) { |m|
#       if Gem::Version.new(m.to_s) < Gem::Version.new('1.1.0')
#         errors.push(" -- \e\[31mVagrant version \"#{m.to_s}\" is too old!\e\[0m\n")
#       else
#         puts " - \e\[32mVagrant \"#{m.to_s}\" works!\e\[0m\n"
#       end
#     }
#   else
#     errors.push(" - \e\[31mProgram: \"vagrant\" not found!\e\[0m\n")
#   end

#   unless errors.empty?
#     puts "The following errors occured:\n#{errors.join()}"
#   end
# end

# desc "Creates a new cookbook."
# task :new_cookbook, :name do |t, args|
#   sh "bundle exec knife cookbook create #{args.name}"
#   sh "bundle exec knife cookbook create_specs #{args.name}"
#   minitest_path = "cookbooks/#{args.name}/files/default/tests/minitest"
#   mkdir_p minitest_path
#   File.open("#{minitest_path}/default_test.rb", 'w') do |test|
#     test.puts "require 'minitest/spec'"
#     test.puts "describe_recipe '#{args.name}::default' do"
#     test.puts "end"
#   end
# end

# desc "Uploads Berkshelf cookbooks to our chef server"
# task :berks_upload do
#   sh "bundle exec berks upload -c config/berks-config.json"
# end

# desc "Creates a Packer Image"
# task :packer_generate do
#     current_dir = File.dirname(__FILE__)
#     nodes = YAML.load_file( "#{current_dir}/packer/packer_images.yml")
#     nodes.each_key do |node_name|
#       include ERB::Util

#       template = File.read("#{current_dir}/packer/template.json.erb")
#       erb = ERB.new(template)
#       File.open("#{current_dir}/packer/#{node_name}.json", "w") do |f|
#         f.write(erb.result(binding))
#       end
#     end
#   end
# end

def shell_out!(cmd) #, cwd)
  command = Mixlib::ShellOut.new(cmd) #, :cwd => cwd)
  command.live_stream = $stdout
  command.run_command
  # command.error! unless command.exitstatus == 0
  raise "Failed to execute '#{cmd}'" unless command.exitstatus == 0
end
