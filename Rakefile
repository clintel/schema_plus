require 'bundler'
Bundler::GemHelper.install_tasks

task :default => :spec

desc 'Run test for adapter whose name is suffix of current Gemfile'
task :spec do
    gemfile = ENV['BUNDLE_GEMFILE']
    fail "BUNDLE_GEMFILE environment variable not set" unless gemfile
    adapter = File.extname(gemfile).sub(/^[.]/, '')
    fail "BUNDLE_GEMFILE filename does not end with .db adapter name" if adapter.empty?
    Rake::Task["#{adapter}:spec"].invoke
end

# work around a bug in rake 10.0.3 with ruby 1.9.2
unless RUBY_VERSION == "1.9.2"
    require 'rdoc/task'
    Rake::RDocTask.new do |rdoc|
    require File.dirname(__FILE__) + '/lib/schema_plus/version'

    rdoc.rdoc_dir = 'rdoc'
    rdoc.title = "schema_plus #{SchemaPlus::VERSION}"
    rdoc.rdoc_files.include('README*')
    rdoc.rdoc_files.include('lib/**/*.rb')
    end
end

require 'rspec/core/rake_task'
%w[postgresql mysql mysql2 sqlite3 sqlserver].each do |adapter|
  namespace adapter do
    RSpec::Core::RakeTask.new(:spec) do |spec|
      spec.rspec_opts = "-Ispec/connections/#{adapter}"
      spec.fail_on_error = true
    end
  end
end

DATABASES = %w[schema_plus_test]
[ 
  { namespace: :postgresql, uservar: 'POSTGRES_DB_USER', defaultuser: 'postgres', create: "createdb -U '%{user}' %{dbname}", drop: "dropdb -U '%{user}' %{dbname}" },
  { namespace: :mysql, uservar: 'MYSQL_DB_USER', defaultuser: 'schema_plus', create: "mysqladmin -u '%{user}' create %{dbname}", drop: "mysqladmin -u '%{user}' -f drop %{dbname}" }
].each do |db|
  namespace db[:namespace] do
    user = ENV.fetch db[:uservar], db[:defaultuser]
    task :create_databases do
      DATABASES.each do |dbname|
        system(db[:create] % {user: user, dbname: dbname})
      end
    end
    task :drop_databases do
      DATABASES.each do |dbname|
        system(db[:drop] % {user: user, dbname: dbname})
      end
    end
  end
end

desc 'Run postgresql, mysql, mysql2 and sqlite3 tests'
task :specs do 
  invoke_multiple(%w[postgresql mysql mysql2 sqlite3], "spec")
end

desc 'Create test databases'
task :create_databases do
  invoke_multiple(%w[postgresql mysql], "create_databases")
end

desc 'Drop test databases'
task :drop_databases do
  invoke_multiple(%w[postgresql mysql], "drop_databases")
end

SQLSERVER_DB_HOST = ENV.fetch('SQLSERVER_DB_HOST', '127.0.0.1')
SQLSERVER_DB_USER = ENV.fetch('SQLSERVER_DB_USER', 'schema_plus')
SQLSERVER_DB_PASSWD = ENV.fetch('SQLSERVER_DB_PASSWD', 'schema_plus')

namespace :sqlserver do
  tsql = "tsql -p 1433 -H #{SQLSERVER_DB_HOST} -U #{SQLSERVER_DB_USER} -P #{SQLSERVER_DB_PASSWD}"

  desc 'Build the SQL Server test databases'
  task :build_databases do
    %x( echo "CREATE DATABASE schema_plus_unittest\ngo" | #{tsql} )
  end

  desc 'Drop the SQL Server test databases'
  task :drop_databases do
    %x( echo "DROP DATABASE schema_plus_unittest\ngo" | #{tsql} )
  end

  desc 'Rebuild the SQL Server test databases'
  task :rebuild_databases => [:drop_databases, :build_databases]
end

def invoke_multiple(namespaces, task)
  failed = namespaces.reject { |adapter|
    begin
      Rake::Task["#{adapter}:#{task}"].invoke
      true
    rescue => e
      warn "\n#{e}\n"
      false
    end
  }
  fail "Failure in: #{failed.join(', ')}" if failed.any?
end
