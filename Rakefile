#!/usr/bin/env rake
begin
  require 'bundler/setup'
  require 'appraisal'
rescue LoadError
  puts 'You must `gem install bundler` and `bundle install` to run rake tasks'
end
require 'bundler/gem_tasks'
require 'rspec/core/rake_task'

namespace :spec do
  RSpec::Core::RakeTask.new(:unit) do |spec|
    spec.pattern = 'spec/unit/*_spec.rb'
    spec.rspec_opts = ['--backtrace']

    rbx = defined?(RUBY_ENGINE) && RUBY_ENGINE == 'rbx'
    jruby = defined?(RUBY_ENGINE) && RUBY_ENGINE == 'jruby'
    if RUBY_VERSION == '1.8.7' && !(rbx || jruby)
      spec.rcov = true
      spec.rcov_opts = %w{--exclude gems\/,spec\/}
    end
  end
  RSpec::Core::RakeTask.new(:integration) do |spec|
    spec.pattern = 'spec/integration/*_spec.rb'
    spec.rspec_opts = ['--backtrace']
  end
  desc "run spec:unit and spec:integration tasks"
  task :all do
    Rake::Task['spec:unit'].execute

    # Travis CI does not expose encrypted ENV variables for pull requests, so
    # do not run integration specs
    # http://about.travis-ci.org/docs/user/build-configuration/#Set-environment-variables
    #
    pull_request = ENV['TRAVIS_PULL_REQUEST'] == 'true'
    ci_build = ENV['TRAVIS'] == 'true'
    if !ci_build || (ci_build && pull_request)
      Rake::Task['spec:integration'].execute
    end
  end
end

task :default => 'spec:all'

Rake::Task[:release].tap do |task|
  task.clear
  task.instance_variable_set :@arg_names, nil
end

desc "Build and release v#{AssetSync::VERSION} to Gemfury"
task :release => :build do
  sh "curl -F package=@pkg/asset_sync-#{AssetSync::VERSION}.gem https://#{ENV.fetch('GEMFURY_TOKEN')}@push.fury.io/#{ENV.fetch('GEMFURY_USERNAME')}/"
  sh "git tag v#{AssetSync::VERSION} -m 'Release #{AssetSync::VERSION}'"
  sh "git push origin v#{AssetSync::VERSION}"
end
