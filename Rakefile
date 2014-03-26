
# @return [Array<String>] The list of the names of the CocoaPods repositories
#         which store a gem.
#
GEM_REPOS = %w[
  CLAide
  CocoaPods
  Core
  Xcodeproj
  cocoapods-docs
  cocoapods-downloader
  cocoapods-podfile_info
  cocoapods-try
]

# Task set-up
#-----------------------------------------------------------------------------#

desc "Clones all the CocoaPods repositories"
task :set_up do
  Rake::Task[:clone].invoke
  Rake::Task[:bootstrap].invoke
end

# Task clone
#-----------------------------------------------------------------------------#

desc "Clones all the CocoaPods repositories"
task :clone do
  repos = fetch_gem_repos

  title "Cloning repositories"
  repos.each do |repo|
    name = repo['name']
    subtitle "\nCloning #{name}"
    url = repo['clone_url']
    if File.exist?(name)
      puts "Already cloned"
    else
      sh "git clone #{url}"
    end
  end
end

# Task bootstrap
#-----------------------------------------------------------------------------#

desc "Runs the Bootstrap task on all the repositories"
task :bootstrap do
  title "Bootstrapping all the repositories"
  Dir['*/'].each do |dir|
    Dir.chdir(dir) do
      subtitle "\nBootstrapping #{dir}"
      if File.exist?('Rakefile')
        has_bootstrap_task = `rake --no-search --tasks bootstrap`.include?('rake bootstrap')
        if has_bootstrap_task
          sh "rake --no-search bootstrap"
        end
      end
    end
  end

  disk_usage = `du -h -c -d 0`.split(' ').first
  puts "\nDisk usage: #{disk_usage}"
end

# Task switch_to_ssh
#-----------------------------------------------------------------------------#

desc "Points the origin remote of all the git repos to use the SSH URL"
task :switch_to_ssh do
  repos = fetch_gem_repos
  title "Setting SSH URLs"
  repos.each do |repo|
    name = repo['name']
    url = repo['ssh_url']
    subtitle(name)
    Dir.chdir(name) do
      sh "git remote set-url origin '#{url}'"
    end
  end
end

# Task pull
#-----------------------------------------------------------------------------#

desc "Pulls all the repositories & updates their submodules"
task :pull do
  title "Pulling all the repositories"
  Dir['*/'].each do |dir|
    Dir.chdir(dir) do
      subtitle "\nPulling #{dir}"
      sh "git pull"
    end
  end
end

# Task set_up_local_dependencies
#-----------------------------------------------------------------------------#

desc "Setups the repositories to use their dependencies from the checkouts (Bundler Local Git Repos feature)"
task :set_up_local_dependencies do
  title "Setting up Bundler's Local Git Repos"
  GEM_REPOS.each do |gem_name|
    sh "bundle config local.#{gem_name} ./#{gem_name}"
  end
end

# Task status
#-----------------------------------------------------------------------------#

desc "Checks the gems which need a release"
task :status do
  title "Checking status"
  dirs = Dir['*/'].map { |dir| dir[0...-1] }

  subtitle "Repositories not in master branch"
  dirs_not_in_master = dirs.reject do |dir|
    Dir.chdir(dir) do
      branch = `git rev-parse --abbrev-ref HEAD`.chomp
      branch == 'master'
    end
  end
  if dirs_not_in_master.empty?
    puts "All repos are on the master branch"
  else
    puts "- #{dirs_not_in_master.join("\n- ")}"
  end

  subtitle "\nRepositories with a dirty working copy"
  dirty_dirs = dirs.reject do |dir|
    Dir.chdir(dir) do
      `git status --short`.chomp
      $?.exitstatus.zero?
    end
  end
  if dirty_dirs.empty?
    puts "All the repositories have a clean working copy"
  else
    puts "- #{dirty_dirs.join("\n- ")}"
  end

  subtitle "\nGems with releases"
  gemspecs = Dir['*/*.gemspec']
  gem_dirs = gemspecs.map { |path| File.dirname(path) }.uniq
  has_pending_releases = false
  gem_dirs.each do |dir|
    Dir.chdir(dir) do
      tag = `git describe --abbrev=0 2>/dev/null`.chomp
      if tag != ''
        commits_since_last_tag = `git rev-list #{tag}..HEAD --count`.chomp
        unless commits_since_last_tag == 0
          has_pending_releases = true
          puts "\n- #{dir}\n  #{commits_since_last_tag} commits since #{tag}"
        end
      end
    end
  end

  unless has_pending_releases
    puts "All the gems are up to date"
  end
end

#-----------------------------------------------------------------------------#
# HELPERS
#-----------------------------------------------------------------------------#

# @return [Array<Hash>] The list of the CocoaPods repositories which contain a
# Gem as returned by the GitHub API.
#
def fetch_gem_repos
  fetch_repos.select do |repo|
    GEM_REPOS.include?(repo['name'])
  end
end

# @return [Array<Hash>] The list of the CocoaPods repositories as returned by
# the GitHub API.
#
def fetch_repos
  require 'json'
  require 'open-uri'
  title "Fetching repositories list"
  url = 'https://api.github.com/orgs/CocoaPods/repos?type=public'
  response = open(url).read
  repos = JSON.parse(response)
  repos.reject! { |repo| repo['name'] == 'LaunchPad' }
  puts "Found #{repos.count} public repositories"
  repos
end

# UI
#-----------------------------------------------------------------------------#

# Prints a title.
#
def title(string)
  puts
  puts "-" * 80
  puts cyan(string)
  puts "-" * 80
  puts
end

def subtitle(string)
  puts green(string)
end

def error(string)
  raise "[!] #{red(string)}"
end

# Colorizes a string to green.
#
def green(string)
  "\033[0;32m#{string}\e[0m"
end

# Colorizes a string to yellow.
#
def yellow(string)
  "\033[0;33m#{string}\e[0m"
end

# Colorizes a string to red.
#
def red(string)
  "\033[0;31m#{string}\e[0m"
end

def cyan(string)
  "\033[0;36m#{string}\033[0m"
end

