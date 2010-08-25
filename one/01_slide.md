!SLIDE
# Building Gems #

## Myron Marston ##

!SLIDE small
# We've got this code in rcommon... #

    @@@ Ruby
    module Validation
      def self.is_url_valid(url)
        # ...
      end
    end

!SLIDE bullets incremental small

# Why would we want to package this in a gem? #

* Standard code sharing mechanism in the ruby community.
* Allows us to version it--now we aren't forced to update all consumers of this code when we change the interface.
* Easy installation and usage from any ruby project.
* Helps us more clearly isolate our code and provide intentional public interfaces.

!SLIDE commandline incremental
# Step 1: Install bundler #

    $ [sudo] gem install bundler --pre
    Successfully installed bundler-1.0.0.rc.6
    1 gem installed
    Installing ri documentation for bundler-1.0.0.rc.6...
    Installing RDoc documentation for bundler-1.0.0.rc.6...

!SLIDE commandline incremental
# Step 2: Generate the project skeleton with bundler #

    $ bundle gem wp_validators
          create  wp_validators/Gemfile
          create  wp_validators/Rakefile
          create  wp_validators/.gitignore
          create  wp_validators/wp_validators.gemspec
          create  wp_validators/lib/wp_validators.rb
          create  wp_validators/lib/wp_validators/version.rb
    Initializating git repo in /home/mmarston/wp_validators

!SLIDE

## What does this give you? ##

!SLIDE small

# Gemfile #

    @@@ Ruby
    source :gemcutter

    # Specify your gem's dependencies in wp_validators.gemspec
    gemspec

!SLIDE bullets incremental

# Having a Gemfile helps in a few ways: #

* Lets contributors easily install all development dependencies with `bundle install`
* In development, locks dependencies to a known working set.

!SLIDE

# Rakefile #

    @@@ Ruby
    require 'bundler'
    Bundler::GemHelper.install_tasks

!SLIDE commandline incremental smaller

## Bundler gives you some default rake tasks ##

    $ rake -T
    (in /home/mmarston/wp_validators)
    rake build    # Build wp_validators-0.0.1.gem into the pkg directory
    rake install  # Build and install wp_validators-0.0.1.gem into system gems
    rake push     # Create tag v0.0.1 and build and push wp_validators-0.0.1.gem to Rubygems

!SLIDE

## You'll need to add your own tasks for running tests. ##
### (We'll do this later) ###

!SLIDE

# .gitignore #

    @@@
    pkg/*
    *.gem
    .bundle

### Ignores gem and bundle artifacts ###

!SLIDE

# lib/wp_validators/version.rb #

    @@@ Ruby
    module WpValidators
      VERSION = "0.0.1"
    end

### Gives you a single canonical place to specify version. ###

!SLIDE smaller

# wp_validators.gemspec #

    @@@ Ruby
    # -*- encoding: utf-8 -*-
    require File.expand_path(
      "../lib/wp_validators/version", __FILE__)

    Gem::Specification.new do |s|
      s.name        = "wp_validators"
      s.version     = WpValidators::VERSION
      s.platform    = Gem::Platform::RUBY
      s.authors     = []
      s.email       = []
      s.homepage    = "http://rubygems.org/gems/wp_validators"
      s.summary     = "TODO: Write a gem summary"
      s.description = "TODO: Write a gem description"

      s.required_rubygems_version = ">= 1.3.6"
      s.rubyforge_project         = "wp_validators"

      s.add_development_dependency "bundler", ">= 1.0.0.rc.6"

      s.files        = `git ls-files`.split("\n")
      s.executables  = `git ls-files`.split("\n").
        map{|f| f =~ /^bin\/(.*)/ ? $1 : nil}.compact
      s.require_path = 'lib'
    end

!SLIDE bullets incremental small

# What is a gemspec? #

* The gemspec contains the metadata for your gem.
* `s.version = WpValidators::VERSION` keeps your `VERSION` constant in sync with the gem version.
* `s.files` and `s.executables` use git to figure out the files.
* You should update `authors`, `email`, `summary` and `description`.

!SLIDE

# Step 3: Setup your test environment. #
### (I like RSpec, but you may opt for Test::Unit, Shoulda, or something else) ###

!SLIDE small

# Add RSpec as a development dependency to the gemspec #

    @@@ Ruby
    s.add_development_dependency "rspec", "~> 1.3.0"

!SLIDE commandline incremental smaller

# Use bundler to install it #

    $ bundle install
    Fetching source index for http://rubygems.org/
    Using bundler (1.0.0.rc.6)
    Enter your password to install the bundled RubyGems to your system:
    Installing rspec (1.3.0)
    Using wp_validators (0.0.1) from source at /home/mmarston/wp_validators
    Your bundle is complete! Use `bundle show [gemname]` to see where a bundled gem is installed.

!SLIDE smaller

# This generates a Gemfile.lock #
## Be sure to check this into git ##

    PATH
      remote: .
      specs:
        wp_validators (0.0.1)

    GEM
      remote: http://rubygems.org/
      specs:
        rspec (1.3.0)

    PLATFORMS
      ruby

    DEPENDENCIES
      bundler (>= 1.0.0.rc.6)
      rspec (~> 1.3.0)
      wp_validators!

!SLIDE

# Add spec task to Rakefile #

    @@@ Ruby
    require 'bundler/setup'
    require 'spec/rake/spectask'
    Spec::Rake::SpecTask.new(:spec)

    task :default => :spec

!SLIDE

# Add spec/spec_helper.rb #

    @@@ Ruby
    require 'bundler'
    Bundler.setup

    require 'wp_validators'

    require 'spec'
    require 'spec/autorun'

    Spec::Runner.configure do |config|
      # future configuration can go here
    end

!SLIDE

# Add an example spec in spec/wp_validators_spec.rb #

    @@@ Ruby
    require 'spec_helper'

    describe WpValidators do
      it "is a module" do
        WpValidators.should be_a(Module)
      end
    end

!SLIDE commandline incremental

# Run the specs to ensure everything is working #

    $ rake
    (in /home/mmarston/wp_validators)
    .

    Finished in 0.001433 seconds

    1 example, 0 failures

!SLIDE

# Step 4: Follow the TDD cycle to develop your code #

!SLIDE smaller
# spec/wp_validators_spec.rb #

    @@@ Ruby
    require 'spec_helper'

    describe WpValidators do
      describe '.is_valid_url?' do
        def be_valid_url
          simple_matcher("a valid url") do |str|
            WpValidators.is_valid_url?(str)
          end
        end

        it 'returns true for a valid http URL' do
          'http://example.com/'.should be_valid_url
        end

        it 'returns true for a valid https URL' do
          'https://example.com/'.should be_valid_url
        end

        it 'returns false for a string that is not a URL' do
          'some string'.should_not be_valid_url
        end

        it 'returns false for an ftp URL' do
          'ftp://example.com/'.should_not be_valid_url
        end
      end
    end

!SLIDE smaller
# lib/wp_validators.rb #

    @@@ Ruby
    require 'uri'

    module WpValidators
      def self.is_valid_url?(string)
        uri = begin
          URI::parse(string)
        rescue URI::InvalidURIError, URI::BadURIError
          return false
        end 

        %w[http https].include?(uri.scheme)
      end 
    end

!SLIDE commandline incremental
# Run the specs... #

    $ rake
    (in /home/mmarston/wp_validators)
    ....

    Finished in 0.00201 seconds

    4 examples, 0 failures

!SLIDE commandline incremental small
# Step 5: Publish your gem #
    $ git tag v0.0.1

    $ git push origin master --tags
    Total 0 (delta 0), reused 0 (delta 0)
    To git@github.com:myronmarston/wp_validators.git
       df5f048..0e7f0be  v0.0.1 -> v0.0.1 

    $ [sudo] gem install gemcutter
    ...
    Successfully installed gemcutter-0.6.1
    1 gem installed
    ...

    $ rake build
    (in /home/mmarston/wp_validators)

    $ gem push pkg/wp_validators-0.0.1.gem 
    Enter your RubyGems.org credentials.
    Don't have an account yet? Create one at http://rubygems.org/sign_up
       Email:   myron.marston@gmail.com
    Password:   
    Signed in.
    Pushing gem to RubyGems.org...
    Successfully registered gem: wp_validators (0.0.1)

!SLIDE small
# Let's use our new gem! #

    $ [sudo] gem install wp_validators
    Successfully installed wp_validators-0.0.1
    1 gem installed
    Installing ri documentation for wp_validators-0.0.1...
    Installing RDoc documentation for wp_validators-0.0.1...

    $ irb
    > require 'rubygems'
     => true 
    > require 'wp_validators'
     => true 
    > WpValidators.is_valid_url?('http://whitepages.com/')
     => true 
    > WpValidators.is_valid_url?('not a url')
     => false 

!SLIDE bullets
# Resources #

* [Gemspec Documentation](http://rubygems.rubyforge.org/rubygems-update/Gem/Specification.html)
* [Bundler Documenation](http://gembundler.com/)
