require 'rake'
require 'bundler/gem_tasks'
require 'rspec/core/rake_task'

def run_in_dummy_app(command)
  success = system("cd spec/dummy && #{command}")
  raise "#{command} failed" unless success
end

task 'default' => 'ci'

desc 'Run all tests for CI'
task 'ci' => 'spec'

desc 'Run all specs'
task 'spec' => 'spec:all'

namespace 'spec' do
  task 'all' => ['drape', 'generators', 'integration']

  def spec_task(name)
    desc "Run #{name} specs"
    RSpec::Core::RakeTask.new(name) do |t|
      t.pattern = "spec/#{name}/**/*_spec.rb"
    end
  end

  spec_task 'drape'
  spec_task 'generators'

  desc 'Run integration specs'
  task 'integration' => ['db:setup', 'integration:all']

  namespace 'integration' do
    task 'all' => ['development', 'production', 'test']

    ['development', 'production'].each do |environment|
      task environment do
        Rake::Task['spec:integration:run'].execute environment
      end
    end

    task 'run' do |_t, environment|
      puts "Running integration specs in #{environment}"

      ENV['RAILS_ENV'] = environment
      success = system('bundle exec rspec spec/integration')

      raise "Integration specs failed in #{environment}" unless success
    end

    task 'test' do
      puts 'Running rake in dummy app'
      ENV['RAILS_ENV'] = 'test'
      run_in_dummy_app 'bundle exec rake'
    end
  end
end

namespace 'db' do
  desc 'Set up databases for integration testing'
  task 'setup' do
    puts 'Setting up databases'

    run_in_dummy_app 'rm -f db/*.sqlite3'

    run_in_dummy_app 'RAILS_ENV=development rails db:environment:set'
    run_in_dummy_app 'RAILS_ENV=development rails db:schema:load db:seed'

    run_in_dummy_app 'RAILS_ENV=production rails db:environment:set'
    run_in_dummy_app 'RAILS_ENV=production rails db:schema:load db:seed'

    run_in_dummy_app 'RAILS_ENV=test rails db:environment:set'
    run_in_dummy_app 'RAILS_ENV=test rails db:schema:load db:seed'
  end
end
