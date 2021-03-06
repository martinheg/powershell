require 'cookstyle'
require 'rubocop/rake_task'
require 'rspec/core/rake_task'
require 'foodcritic'
require 'kitchen'

require_relative 'tasks/maintainers'

# Style tests. Rubocop and Foodcritic
namespace :style do
  desc 'Run Ruby style checks'

  RuboCop::RakeTask.new(:ruby) do |task|
    task.options << '--display-cop-names'
  end

  desc 'Run Chef style checks'
  FoodCritic::Rake::LintTask.new(:chef) do |t|
    t.options = {
      fail_tags: ['any']
    }
  end
end

desc 'Run all style checks'
task style: ['style:chef', 'style:ruby']

# Rspec and ChefSpec
desc 'Run ChefSpec examples'
RSpec::Core::RakeTask.new(:spec)

# Integration tests. Kitchen.ci
namespace :integration do
  desc 'Run Test Kitchen with Vagrant'
  task :vagrant do
    Kitchen.logger = Kitchen.default_file_logger
    Kitchen::Config.new.instances.each do |instance|
      instance.test(:always)
    end
  end

  desc 'Run Test Kitchen with cloud plugins'
  task :cloud do
    run_kitchen = true
    if ENV['TRAVIS'] == 'true' && ENV['TRAVIS_PULL_REQUEST'] != 'false'
      run_kitchen = false
    end

    if run_kitchen
      Kitchen.logger = Kitchen.default_file_logger
      @loader = Kitchen::Loader::YAML.new(project_config: './.kitchen.cloud.yml')
      config = Kitchen::Config.new(loader: @loader)
      config.instances.each do |instance|
        instance.test(:always)
      end
    end
  end
end

desc 'Run all tests on Travis'
task travis: ['style', 'spec', 'integration:cloud']

# Default
task default: ['style', 'spec', 'integration:vagrant']

begin
  require 'github_changelog_generator/task'
  require 'stove'
  require 'stove/cookbook/metadata'

  GitHubChangelogGenerator::RakeTask.new :changelog do |config|
    config.future_release = "v#{Stove::Cookbook::Metadata.from_file('./metadata.rb').version}"
    config.pulls = true
    config.issues = false
    config.user = 'chef-cookbooks'
    config.project = 'powershell'
    # require 'pry'; binding.pry
  end
rescue LoadError
  puts 'github_changelog_generator is not available.' \
       ' gem install github_changelog_generator to generate changelogs'
end
