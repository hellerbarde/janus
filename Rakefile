ROOT_PATH = File.expand_path(File.join(File.dirname(__FILE__)))
$: << File.join(ROOT_PATH, 'janus', 'ruby')

require 'janus'
require 'fileutils'
include Janus
require 'rbconfig'
# Some versions of Ruby on Windows apparently don't have ln_s implemented
is_windows = (RbConfig::CONFIG['host_os'] =~ /mswin|mingw|cygwin/)

desc "link ViM configuration files."
task :link_vim_conf_files do
  %w[ vimrc gvimrc ].each do |file|
    dest = expand("~/.#{file}")
    unless File.exist?(dest)
      abs_path = expand("../janus/vim/#{file}", __FILE__)
      if is_windows
        #dest.gsub!("/", "\\")
        #abs_path.gsub!("/", "\\")
        #system("mklink #{dest} #{abs_path} 1>&2")
	cp(abs_path, dest)
      else
        ln_s(abs_path, dest)
      end
    end
  end
end

namespace :dev do
  desc "Update submodules"
  task :update_submodules do
    sh "git submodule foreach git pull origin master"
  end

  # Taken from RefineryCMD
  # https://github.com/resolve/refinerycms/blob/master/core/lib/tasks/refinery.rake
  desc 'Removes trailing whitespace across the entire application.'
  task :whitespace do
    require 'rbconfig'
    if RbConfig::CONFIG['host_os'] =~ /linux/
      sh %{find . -name '*.*rb' -o -name '*.*vim' -exec sed -i 's/\t/ /g' {} \\; -exec sed -i 's/ *$//g' {} \\; }
    elsif RbConfig::CONFIG['host_os'] =~ /darwin/
      sh %{find . -name '*.*rb' -o -name '*.*vim' -exec sed -i '' 's/\t/ /g' {} \\; -exec sed -i '' 's/ *$//g' {} \\; }
    else
      puts "This doesn't work on systems other than OSX or Linux. Please use a custom whitespace tool for your platform '#{RbConfig::CONFIG["host_os"]}'."
    end
  end
end

desc "Create necessary folders."
task :folders do
  Janus::VIM.folders.each do |folder|
    mkdir_p folder
  end
end

task :update do
  puts "Cleaning the janus folder"
  `git clean -xdf -- janus 2>&1`
  `git ls-files --exclude-standard --others -- janus`.split("\n").each do |untracked|
    FileUtils.rm_rf File.expand_path(untracked.chomp, File.dirname(__FILE__))
  end

  puts "Pulling latest changes"
  `git pull`

  puts "Cleaning the janus folder"
  `git clean -xdf -- janus 2>&1 /dev/null`
  `git ls-files --exclude-standard --others -- janus`.split("\n").each do |untracked|
    FileUtils.rm_rf File.expand_path(untracked.chomp, File.dirname(__FILE__))
  end

  puts "Synchronising submodules urls"
  `git submodule sync`

  puts "Updating the submodules"
  `git submodule update --init`
end

task :install => [:folders, :link_vim_conf_files] do
  # Dummy task, real work is done with the hooks.
end

desc "Install or Update Janus."
task :default do
  sh "rake update"
  sh "rake install"
end
