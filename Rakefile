unless File.exist?("./.gemfile")
  warn <<-MESSAGE
=============================================================================
You must set the version of rails you want to run against. The simplest way
to accomplish this is to install thor (if you don't already have it) and run:

    thor rails:use 3.0.6

You can use any of the following versions/branches:

  3.0.0 .. 3.0.6
  master
  3-0-stable

See the README_DEV.md file for more information.
=============================================================================

MESSAGE
  exit 1
end

require 'pathname'
ENV["BUNDLE_GEMFILE"] ||= begin
                            version = if File.exist?("./.gemfile")
                                        File.read("./.gemfile").chomp
                                      else
                                        "rails-3.0.6"
                                      end
                            File.expand_path("../gemfiles/#{version}", __FILE__)
                          end
puts "Using gemfile: #{ENV["BUNDLE_GEMFILE"].gsub(Pathname.new(__FILE__).dirname.to_s,'').sub(/^\//,'')}"
require "bundler"
Bundler.setup
Bundler::GemHelper.install_tasks

require 'rake'
require 'yaml'

require 'rake/rdoctask'
require 'rspec'
require 'rspec/core/rake_task'
require 'cucumber/rake/task'

task :cleanup_rcov_files do
  rm_rf 'coverage.data'
end

desc "Run all examples"
RSpec::Core::RakeTask.new(:spec) do |t|
  t.rspec_opts = %w[--color]
end

Cucumber::Rake::Task.new(:cucumber)

namespace :spec do
  desc "Run all examples using rcov"
  RSpec::Core::RakeTask.new :rcov => :cleanup_rcov_files do |t|
    t.rcov = true
    t.rcov_opts =  %[-Ilib -Ispec --exclude "gems/*,features"]
    t.rcov_opts << %[--text-report --sort coverage --no-html --aggregate coverage.data]
  end
end

namespace :cucumber do
  desc "Run cucumber features using rcov"
  Cucumber::Rake::Task.new :rcov => :cleanup_rcov_files do |t|
    t.cucumber_opts = %w{--format progress}
    t.rcov = true
    t.rcov_opts =  %[-Ilib -Ispec --exclude "gems/*,features"]
    t.rcov_opts << %[--text-report --sort coverage --aggregate coverage.data]
  end
end

namespace :generate do
  desc "generate a fresh app with rspec installed"
  task :app do |t|
    unless File.directory?('./tmp/example_app')
      sh "bin/rails new ./tmp/example_app"
      bindir = File.expand_path("gemfiles/bin")
      Dir.chdir("./tmp/example_app") do
        sh "ln -s #{bindir}"
      end
    end
  end

  desc "generate a bunch of stuff with generators"
  task :stuff do
    in_example_app "rake rails:template LOCATION='../../templates/generate_stuff.rb'"
  end
end

def in_example_app(command)
  Dir.chdir("./tmp/example_app/") do
    Bundler.with_clean_env do
      sh command
    end
  end
end

namespace :db do
  task :migrate do
    in_example_app "rake db:migrate"
  end

  namespace :test do
    task :prepare do
      in_example_app "rake db:test:prepare"
    end
  end
end

desc "run a variety of specs against the generated app"
task :smoke do
  in_example_app "rake rails:template --trace LOCATION='../../templates/run_specs.rb'"
end

desc 'clobber generated files'
task :clobber do
  rm_rf "pkg"
  rm_rf "tmp"
  rm    "Gemfile.lock" if File.exist?("Gemfile.lock")
end

namespace :clobber do
  desc "clobber the generated app"
  task :app do
    rm_rf "tmp/example_app"
  end
end

desc "Push docs/cukes to relishapp using the relish-client-gem"
task :relish, :version do |t, args|
  raise "rake relish[VERSION]" unless args[:version]
  sh "relish push rspec/rspec-rails:#{args[:version]}"
end

task :default => [:spec, "clobber:app", "generate:app", "generate:stuff", :smoke, :cucumber]
