# Development Environment

These notes correspond to the installation lectures on the course web site and 
have been annotated to reflect changes as of **2017-04-16** to those videos.

## Basic Requirements

* Chrome Browser
* Firefox Browser
* Git (os-specific)
* rbenv (os-specific)
* ruby (os-specific)
* PostgreSQL (os-specific)
* MongoDB (os-specific)
* Node.js (os-specific)
* ImageMagick (os-specific)
* PhantomJS (os-specific)
* Sublime Text Editor (os-specific)
* Rails

Much of the installation instructions included here are based on a
very complete set of instructions at
[gorails.com/setup](https://gorails.com/setup) and the other courses
in this specialization. Please reference the material on these sites
in addition to these course instructions. Both will provide a
complementary understanding on what is necessary for the Capstone
project.

**Note**:  For students utilizing Windows 10 as his/her development environment,
we will not be utilizing the instuctions provided by `gorails.com`. Those
instructions are reliant upong a beta version of the Windows Subsystem for Linux 
(WSL), which imposes some integration and compability issues for the Capstone.
Instead, we will leverage the complete set of instructions provided in 
[Course 1](http://www.coursera.org/learn/ruby-on-rails-intro/lecture/kWeIk/software-installation-for-windows-users)
plus some updates and extensions.

## Install Chrome Browser

The following Google Help page provides detailed information on how to install
Chrome for Windows, Mac, and Linux

* [Download and install Google Chrome](https://support.google.com/chrome/answer/95346?hl=en&) 

## Install Mozilla/Firefox Browser

Although Mozilla Firefox can be used as a default browser and development,
we will be specifically using it as an alternative testing environment.
Unfortunately this browser is going through some major release changes and the 
Selenium test driver has not fully caught up with the latest release. The 
extended support release (ESR) version should be used -- which is 45.5.0esr
at this time.

1. [Download and install Mozilla Firefox ESR](https://www.mozilla.org/en-US/firefox/organizations/all/)

2. Verify version `45.x.x esr` installed by browsing to `about:healthreport` and 
looking at the version and update channel under `Vital Stats`.

    ```text
    # about:healthreport

    Vital Stats

        version45.5.0
        update channelesr
        updates automatic
    ```

    * **2017-04-26 Note:** Firefox 45 ESR is at version 45.8.0. It
    is not critical that you install Firefox (we are using it as
    an alternate browser for testing), but if you do download it
    the version must  be in the 45.x series. Feel free to skip this
    step if you do not plan to test with Firefox.  You can always
    decide later.


## Operating System Specific Instructions

See the following instructions for operating system-specific instructions for various 
tools.

* [Mac OS](./macos_dev_env.md)
* [Windows](./windows_dev_env.md)
* [Docker/Compose Debian](./docker_debian_env.md)

## Install `rails` gem

Download/install the `rails` gem used to create a full application
server instance. 

1. Install the `rails` gem using the Ruby gem installer. This will take several
minutes, so plan a break while the installation is in progress.

    ```shell
    $ gem install rails -v 4.2.6 --no-ri --no-doc
    ```

2. If you are using `rbenv`, add the `rails` commands to the shell using
`rbenv rehash`

    ```shell
    $ rbenv rehash
    ```

3. Verify the version installed

    ```shell
    $ rails -v
    Rails 4.2.6
    ```

## Install the `rails-api` gem

Download/install the `rails-api` gem used to create a 'mostly-api' version of 
the Rails server. This gem is now officially part of Rails in Rails 5.
We will use it in this course to demonstrate its purpose and how to re-enable
what we need from what it disables by default.

1. Install the `rails-api` gem using the Ruby gem installer.

    ```shell
    $ gem install rails-api -v 0.4.0 --no-ri --no-doc
    Fetching: railties-5.0.1.gem (100%)
    Successfully installed railties-5.0.1
    Fetching: rails-api-0.4.0.gem (100%)
    Successfully installed rails-api-0.4.0
    2 gems installed
    ```

    **2017-07-11**: Unfortunately railties 5.0.x will get installed with it.

    ```shell
    $ rails-api -v
    Rails 5.0.1
    ```

    **2017-07-11**: Uninstall railties 5.0.x

    ```shell
    $ gem uninstall railties -v 5.0.1
    Successfully uninstalled railties-5.0.1
    ```

2. If you are using `rbenv`, add `rails-api` commands to the shell
using `rbenv rehash`

    ```shell
    $ rbenv rehash
    ```

3. Verify the `rails-api` command is accessible and reports version `4.2.6`.

    ```shell
    $ rails-api -v
    Rails 4.2.6
    ```

## Install the bundler

This is used to conveniently install gems defined by the Rails Gemfile.

```shell
$ gem install bundler --no-ri --no-doc
```


## Summary

That completes the bulk of the software installations for the entire course.
Many of the things installed will be used within the module as we develop
and deploy a quick end-to-end API application. The remaining items installed
will be used in the modules to come. By mentioning them here we have everything
we need installed to checkout samples of the solution repository to test
our environment.

## Early Sanity Check

You can perform an early sanity check on your development environment by 
following the following build steps.

1. Clone a copy of the course example

    ```shell
    $ git clone https://github.com/jhu-ep-coursera/capstone_demoapp.git module1
    ```

2. Checkout the tag at the end of module #1 and create a working branch for module 1

    ```shell
    $ cd module1
    $ git checkout module2.start
    $ git checkout -b module1
    ```

3. Run bundle to download the gems in the Gemfile

    ```shell
    $ bundle
    ```

4. Create the database for both the development and test profiles.

    ```shell
    $ bundle exec rake db:create
    ```

    * **Note**: If you get a permission error with the username set to nil, assign
    the POSTGRES_USER environment variable to be `postgres`

5. Migrate the database schema for the development and test profiles.

    ```shell
    $ bundle exec rake db:migrate
    $ bundle exec rake db:migrate RAILS_ENV=test
    ```

6. Run the unit tests for the example application

    ```shell
    $ bundle exec rake 
    ```

    You can optionally use later tags in the course, but with each 
    module comes additional complexity -- so you may want to limit 
    how far you test in the future until you are comfortable with 
    your current progres in the course. 
