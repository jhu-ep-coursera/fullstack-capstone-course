# MAC OS X (10.11) Setup

The following instructions contain MacOS-specific extensions to the
[overall software installation requirements](./dev_env.md) These
are based on a very complete set of instructions at
[gorails.com/setup](https://gorails.com/setup). Please reference
the material on that site and these instructions will highlight
what was necessary for the Capstone project.


## Install Homebrew

1. Run the following command at the terminal to install Homebrew

    ```shell
    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    ```

    The download and installation should complete and the `brew` command
    should be able to display a version

    ```shell
    $ brew --version
    Homebrew 1.1.1
    Homebrew/homebrew-core (git revision 4b96; last commit 2016-11-26)
    ```

    **2017-04-16 Note**: Version was `1.1.12`

## Install rbenv

rbenv is used to install and manage versions of Ruby.

* Select Ruby 2.3.3 dropdown


1. Enter the following command to install the Ruby version manager. 

    ```
    $ brew install rbenv ruby-build
    ```

2. Add rbenv to your environment

    ```shell
    $ echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.bash_profile
    $ source ~/.bash_profile
    $ rbenv --version
    rbenv 1.1.0
    ```

## Install Ruby

It is worth noting that the site points out that MacOS comes with its own version of Ruby
that is not sufficient for use in this course. These instructions will not only update 
the version of Ruby but also install a new way of managing Ruby.

```shell
$ rbenv install 2.3.3
$ rbenv global 2.3.3
$ ruby -v
$ ruby --version
ruby 2.3.3p222 (2016-11-21 revision 56859) [x86_64-darwin15]
```

**2017-04-16 Note**: Ruby 2.3 Version listed is `2.3.3`



## Install Git

The OS X operating system ships with a version of git. You can update it to a recent copy using Homebrew.

1. Install an up to date copy of git using Homebrew.

    ```shell
    $ brew install git
    $ source ~/.bash_profile
    $ git --version
    git version 2.12.2
    ```

2. Configure personal aspects of git.

    ``` shell
    $ git config --global color.ui true
    $ git config --global user.name "YOUR NAME"
    $ git config --global user.email "YOUR@EMAIL.com"
    $ ssh-keygen -t rsa -C "YOUR@EMAIL.com"
    ```

3. Register generated public key with Github and verify the 
results with the following command

    ```shell
    $ ssh -T git@github.com
    Warning: Permanently added the RSA host key for IP address 'xxx.xx.xxx.xxx' to the list of known
    hosts.
    Hi YOURNAME! You've successfully authenticated, but GitHub does not provide shell access.
    ```

## (Optional) Install PostgreSQL

The deployment environment (Heroku) uses PostresSQL so it makes sense to keep 
the development environment as close to production as possible without incurring 
too much pain. Students do have the option of:

* using SQLite for development and deploying to PostreSQL, 
* using PostgresSQL for both development and deployment, or 
* flipping between the two from time-to-time. 

The most noticeable functional difference between the two environments is that 
SQLite does not enforce foreign key constraints and PostgreSQL (and just about 
every other RDBMS on this earth) does. Using SQLite during a lengthy development 
cycle, when our target environment uses PostgreSQL, will likely defer errors 
that will eventually surface when the student's code is migrated 
(i.e., we declared a foreign key before the table/column existed that could 
satisfy the constraint).

1. To install PostgreSQL

    ```
    $ brew install postgresql
    ```

    **2017-04-16 Note**: Version is `9.6.2`

2. To manually start PostgreSQL in the background, the simplest way I found was to execute the following.

    ```
    $ brew services start postgresql
    ```

3. To later stop postgres, use

    ```
    $ brew services stop postgresql
    ```

## Install MongoDB

1. You can install MongoDB manually 
As the [MongoDB Installation Instructions](https://docs.mongodb.com/v3.0/tutorial/install-mongodb-on-os-x/)
state, you can install MongoDB manually or also with brew.

    ```
    $ brew install mongodb
    ```

    **2017-04-16 Note**: Version is `v3.4.3`

2. If you installed using brew, start MongoDB using `brew services`.

    ```
    $ brew services start mongodb
    ```

3. If you installed using brew, stop MongoDB using `brew services`.

    ```
    $ brew services stop mongodb
    ```

## Install Node.js

You may need to add a javascript runtime to your Gemfile in order for bootstrap's LESS files to compile to CSS

1. Install Node.js using Homebrew.

    **2017-04-16 Note**: Version is `v7.9.0`

    ```shell
    $ brew install node
    $ node --version
    v7.9.0
    ```
## Install ImageMagick

This native application will be primarily used to create images of
different scale sizes from an original. Please use version 6.9.x
as the newer version 7 has changed commands and has broken the
interface between the MiniMagick Ruby gem and the native application.
When I last installed with bew -- it supplied version 6.9.x.

1. Install Imagemagick using Homebrew.

    **Note**: Version 6 is no longer the default. This legacy wrapping 
    looks promising to use. 

    ```shell
    $ brew install imagemagick@6
    $ echo 'export PATH="$PATH:/usr/local/opt/imagemagick@6/bin"' >> ~/.bash_profile
    $ source ~/.bash_profile
    ```

2. Verify the installation by executing the following command

    ```shell
    $ mogrify --version
    Version: ImageMagick 6.9.8-3 Q16 x86_64 2017-03-25 http://www.imagemagick.org
    Copyright: © 1999-2017 ImageMagick Studio LLC
    License: http://www.imagemagick.org/script/license.php
    Features: Cipher DPC Modules 
    Delegates (built-in): bzlib freetype jng jpeg ltdl lzma png tiff xml zlib
    ```

    Some gems interface with ImageMagic using libraries. Our Minimagick
    gem uses the command line interface to execute the `mogrify` command.
    Version 7 of Imagemagick appears to have changed that command and if
    you see version 7 being installed or cannot locate the `mogrify` command
    -- we will have problems later in your development environment getting
    image manipulation to work seemlessly


## Install PhantomJS

In this section we are going to install the driver for headless 
web UI testing.

1. Install PhantomJS using Homebrew.

    ```shell
    $ brew install phantomjs
    ```

2. Verify the installation by executing the following command

    ```shell
    $ phantomjs --version
    2.1.1
    ```

## Install ChromeDriver

Selenium testing requires that we download and install a driver for 
Chrome and place it in our path. 

```text
Unable to find the chromedriver executable. Please download the server from
http://chromedriver.storage.googleapis.com/index.html and place it somewhere on your PATH. More info at
https://github.com/SeleniumHQ/selenium/wiki/ChromeDriver.
```

1. Install `chromedriver` using Homebrew.

    ```shell
    $ brew install chromedriver
    ```

    **2017-04-16 Note**: Current version of chromedriver is 2.29 but 
    needs to match the version of the Chrome browser you have installed.

2. Test that chrome is available in the PATH

    ```shell
    $ chromedriver --version
    ChromeDriver 2.25.426935 (820a95b0b81d33e42712f9198c215f703412e1a1)
    ```

## (Optional) Install Sublime Text 3

Any editor is fine for use with the class. The instructor will switch between 
`vi` and `Sublime Text`, depending on the work being performed. You may 
also install and use Sablime Text of you wish by either downloading and installing
the binary or using Homebrew -- which will be shown here.

1. Install Sublime Text using Homebrew

    ```shell
    $ brew install sublime-text
    ```

2. Verify the installation by executing the following command

    ```shell
    $ subl --version
    Sublime Text Build 3126
    ```

3. Verify the editor starts

4. (Optional) Install some Angular templates 
[published by John Papa](https://johnpapa.net/angularjs-snippets-for-sublime-visual-studio-and-webstorm/). 
Even if you do not use them, they are a good example of how to create other templates. The 
easiest way to download them as a set is by checking out the entire styleguide Git repo
and copying the contents of the `sublime-angular-snippets` to the directly listed in the
following [Sublime 3 Documentation Page](https://packagecontrol.io/packages/Sublime%20Text%203%20Snippets)

    ```shell
    git clone https://github.com/johnpapa/angular-styleguide.git
    cp angular-styleguide/a1/assets/sublime-angular-snippets/*  ...
    ```

    * On MacOS, I copied them to 
    `/Users/jim/Library/Application Support/Sublime Text 3/Packages/Sublime Text 3 Snippets`.
    Like with Windows -- the base `.../Packages` directory was created for me once
    the editor was launched the first time. The `Sublime Text 3 Snippets` did not exist 
    and had to be created.

5. Test the installation of the templates by

    * restarting the editor
    * typing the letters `ng` into one of the editor files
    * hitting control-spacebar and selecting from the dropdown

    We should see a list of templates from what we copied over. If
    we select one -- it will expand the template into our source
    file. We can modify the original templates and add new ones to
    suite our tastes.


## Summary 

We have now finished installing software that may have commands
that are unique to MacOS. At this point in time we can return to 
the generic instructions to complete our full stack development 
environment setup.
