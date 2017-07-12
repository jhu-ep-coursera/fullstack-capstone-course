# MS Windows (10) Setup

The following instructions contain MS Windows-specific extensions to the
[overall software installation requirements](./dev_env.md). These
provide extensions and updates to the complete set of instructions detailed within 
[Course 1](http://www.coursera.org/learn/ruby-on-rails-intro/lecture/kWeIk/software-installation-for-windows-users). 

## Install the RailsInstaller package for Windows

The RailsInstaller was used to install Ruby, Rails, and Git during Course #1 and 
we will stick with that. The RailsInstaller tends to be slightly behind in its 
supported version of Ruby, but should not matter for this course. 

**2017-04-16 Update**: The current installer is Ruby 2.3 (great!),
but also includes Rails 5.0. The course example is using Rails 4.2.6
to try to stay consistent with previous courses in the specialization.
Please download and install using the RailsInstaller, but downgrade
your version of Rails prior to completing.

1. Visit the [RailsInstaller site](http://railsinstaller.org/en)
2. Select the `Windows Ruby 2.3` package for download and launch the executable
3. Leave the defaults on the setup page and select `Install`  
4. As part of a successful installation, the installer will ask for
the name and email of the student to complete its Git configuration. What we 
supply does not matter, but these values will be part of every commit we make
and helps others identify who made that commit.

    ```shell
    name > first last
    Setting user.name to first last

    email > first.last@example.com
    Setting user.email to first.last@example.com
    ```

5. A public/private ssh keypair will be generated for git and placed in
`C:\Users\<login id>\.ssh`. That can be optionally used to register your 
development environment with Github.

6. Verify correct communication with Internet gem repositories. If we
receive an SSL error as shown below -- follow the corrective setup steps.

    **2017-04-16 Update**: The following is not an issue with the Ruby 2.3 
    installer. This was only true with the Ruby 2.2/Rails 2.4.6 installer.
    Go ahead and run `gem update --system --no-ri --no-doc` without error
    when using Ruby 2.3 installer. Skip to step 7 in this section after this.

    **Note**: Rails on Windows installs with SSL verification enabled
    to assure that only trusted sites are used for downloads. However,
    the installation is missing a few things to complete that verification
    and must be patched before we continue.

    ```shell
    $ gem update --system --no-ri --no-doc

    c:\Users\jim>gem update --system --no-ri --no-doc
    ERROR:  While executing gem ... (Gem::RemoteFetcher::FetchError)
        SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://api.rubygems.org/specs.4.8.gz)
    ```

    * (Optional) Review the description of the problem and solution on the following 
    [web page](http://guides.rubygems.org/ssl-certificate-update/#background). The 
    following steps are gisted from that page.

    * Locate where Rubygems is located and the SSL certs are stored. 

        * Run the following command to locate where Rubygems is located.
        Rubygems is a library packaging system that is used to download
        gems required for your applications to run.

            ```shell
            > gem which rubygems
            C:/RailsInstaller/Ruby2.2.0/lib/ruby/2.2.0/rubygems.rb
            ```

        * Feed that result into the following command, changing the
	back-slashes to forward-slashes. The file explorer should
	appear.

            ```shell
            > start C:\RailsInstaller\Ruby2.2.0\lib\ruby\2.2.0\rubygems
            ```
	    * **Note**: This was just a techy way of simply opening
	    the file explorer to a specific directory

        * Locate the `ssl_certs` directory populated with `.pem` files.
        Use that directory in the following step.

    * Download the 
    [PEM](https://raw.githubusercontent.com/rubygems/rubygems/master/lib/rubygems/ssl_certs/index.rubygems.org/GlobalSignRootCA.pem) 
    with the new trust certificate and store in the `ssl_certs`
    directory with all the other certs as a `.pem` file.

    * Run the following command again to verify success.  The primary
    task is to verify the certificate works. However, when successful,
    this command also fixes some SSL issues we would have encountered
    later.

        ```shell
        > gem update --system --no-ri --no-doc
        Updating rubygems-update
        Fetching: rubygems-update-2.6.8.gem (100%)
        Successfully installed rubygems-update-2.6.8
        Installing RubyGems 2.6.8
        RubyGems 2.6.8 installed

            ...

        RubyGems installed the following executables:
                C:/RailsInstaller/Ruby2.2.0/bin/gem

        RubyGems system software updated
        ```

7. Add a root trust store file for bundler and other libraries to use.
One of those libraries is 
This will be similar to what was done previously except that it does not 
matter where we store it -- but we must reference it through an environment
variable called `SSL_CERT_FILE`. We will be unable to demonstrate the 
error caused by this missing step until later in the course, but it will
be good to get this out of the way at this time.

    * From the following
    [page](https://gist.github.com/fnichol/867550) 
    that describes the error, save the following 
    [PEM](http://curl.haxx.se/ca/cacert.pem) 
    to any directory

    * Assign the following environment variable to reference that 
    file. `SSL_CERT_FILE` is used to reference the public certs of 
    trusted certificate authorities. It will be used by the bundler
    and other SSL clients to expand their list of places our Ruby 
    application can safelky communicate with. If you here if solutions
    that turn off SSL host verification, they have not gone far enough
    in registering this file.

        ```shell
        set SSL_CERT_FILE=C:\RailsInstaller\...\ssl_certs\cacert.pem
        ```

        If done correctly, the following should echo the path specified

        ```shell
        >ruby -e "puts ENV['SSL_CERT_FILE']"
        C:\RailsInstaller\...\ssl_certs\cacert.pem
        ```

    * Make the environment variable persistent by adding it to the 
    Environment Variables in your System Properties. 

    * Revisit this section once more of Rails and dependencies are specified
    if we get additional SSL errors running the `bundle` command.

8. Remove Rails 5

The RailsInstaller installation will automatically add the rails and 
railties 5 gems (and dependencies) to the installation. Remove these
and we will install 4.2.6 later.


```shell
gem uninstall rails
gem uninstall railties
```

If you run into an error with `bcrypt` later, try the following
(https://github.com/codahale/bcrypt-ruby/issues/142)

```shell
$ gem uninstall bcrypt
$ gem uninstall bcrypt-ruby
$ gem install bcrypt --platform=ruby
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

1. This [PostgreSQL downloads page](https://www.postgresql.org/download/windows/)
provides the Windows install executables from two sources: EnterpriseDB and BigSQL. 
Either should work. I went with BigSQL since it mentioned the words "developer-friendly"
and found that nothing needed to be done after the installer completed.

2. Navigate through the installer set-up screens by leaving the defaults

    **2017-04-16 Note**: You now must manually select the pgDevOps
    option if you want the Web UI installed. The Web UI is not required.

3. When prompted - provide a database server password. This is a local development
database instance and we will be performing various management commands via `rake`
from our application (e.g., `rake db:create`). We will want to store this password
in an environment variable so that we do not check that value into Git with our
source code.

    * Make the environment variable persistent by adding it to the 
    Environment Variables in your System Properties using the name
    `POSTGRES_PASSWORD`.

4. Verify that the postgresql commands are in `$PATH`. **Note**: BigSQL auotmatically
makes the binaries available in the path. If you installed EnterpriseDB, then
you must add `C:\Program Files\PostgreSQL\9.6\bin` you your ENV PATH before performing
the following steps.

    ```shell
    $ psql --version
    psql (PostgreSQL) x.x.x
    ```

5. Verify a connection to the database server can be made

    ```shell
    $ psql -U postgres
    psql (x.x.x)
    postgres=#
    ```

  **Note** to exit out of the postgres SQL prompt, type `\q` 

## Install MongoDB

1. Students can find manual installation steps for MongoDB using the 
[Install MongoDB on Windows page](https://docs.mongodb.com/v3.0/tutorial/install-mongodb-on-windows/)

2. Navigate through the installer set-up screens by leaving the defaults

3. Update `$PATH` to include the mongodb commands.  In a `File Explorer` 
Window: 
    * Right click on `this PC` within the left pane
    * Select `Properties`
    * Select `Advanced System Setings`
    * Update `$PATH` with the absolute path to the mongodb bin directory, i.e. `C:\Program Files\MongoDB\Server\<version>\bin`

4. Verify that the mongodb commands are in `$PATH`

    ```shell
    $ mongod --version
    db version vx.x.x
    ...
    ```

5. Create a default data directory from within the Command Prompt 

    ```shell
    $ md \data\db
    ```

6. Start MongoDB  

    ```
    $ mongod
    MongoDB Starting : ...
    ...
    waiting for connections on port 27017
    ```

7. To stop MongoDB - from within the command prompt that has mongodb running, 
press `Control+C`

## Install Node.js

Students may need to add a javascript runtime to their Gemfile in order for
bootstrapping LESS files that compile to CSS

1. Install `Node.js` using the Windows Installer provided by the 
[Node.js dowloads page](https://nodejs.org/en/download/) 

2. Navigate through the installer set-up screens by leaving the defaults

3. Update `$PATH` to include the node commands. In a `File Explorer` 
Window: 
    * Right click on `this PC` within the left pane
    * Select `Properties`
    * Select `Advanced System Settings`
    * Update `$PATH` with the absolute path to the nodejs directory, i.e. `C:\Program Files\nodejs`

4. Verify that the mongodb commands are in `$PATH`

    ```shell
    $ node --version
    v6.9.1
    ```

    **2017-04-16 Note**: Version was `v6.10.2`
    **2017-07-11 Note**: Version was `v6.11.1`

## Install ImageMagick

This native application will be primarily used to create images of different
scale sizes from an original. Please use version 6.9.6 Q8 DLL for the product
as the newer version 7 has changed commands and has broken the interface between
the MiniMagick Ruby gem and the native application.

1. Download the the correct version for your platform from the 
[ImageMagick download site](https://www.imagemagick.org/download/binaries/). 
Download version 6.9.6, Q8, DLL 

    **2017-04-16 Note**: Version was `ImageMagick-6.9.8-3-Q8-x64-dll.exe`
    **2017-07-11 Note**: Version was `ImageMagick-6.9.8-10-Q8-x64-dll.exe`

2. Run the downloaded installer and answer the prompts with the default answers.

3. Verify the installation by executing the following command in a new CMD window

    ```shell
    C:\Users\jim>mogrify --version
    Version: ImageMagick 6.9.8-3 Q8 x64 2017-03-25 http://www.imagemagick.org
    Copyright: Copyright (C) 1999-2015 ImageMagick Studio LLC
    License: http://www.imagemagick.org/script/license.php
    Visual C++: 180040629
    Features: Cipher DPC Modules OpenMP
    Delegates (built-in): bzlib cairo flif freetype jng jp2 jpeg lcms lqr openexr pangocairo png ps rsvg tiff webp xml zlib
    ```

## Install PhantomJS

In this section we are going to install the driver for headless 
web UI testing.

1. [Download](https://bitbucket.org/ariya/phantomjs/downloads)
version `2.1.1` of the PhantomJS driver for your platform.

2. Unzip the downloaded archive. The specific location of where you 
extract it to is not important as long as you can express a valid 
path to the `phantomjs` executable and the path does not have spaces. 

    **Note**: Although the command-line check worked with
    the folder below "Program Files" once added to the path,
    the space character within the path seemed to throw off
    Capybara at runtime. I had success creating a directory
    `c:\apps` (without spaces) and storing below there.


    ```shell
    C:\apps\phantomjs-2.1.1-windows\bin\phantomjs
    ```

3. Add the `phantomjs` executable to your PATH. This must be 
accessible from where you execute your `rails` application
so make it persistent using the Windows ENV.

    * Open a new CMD window after adding `phantomjs` to the 
    PATH using Windows ENV.

    ```shell
    > phantomjs --version
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

1. [Download](https://sites.google.com/a/chromium.org/chromedriver/downloads)
a recent version of ChromeDriver.

    **2017-04-16 Note**: Verson `2.29` supports browser versions v56-58. Make 
    sure to download version that is appropriate for your version of the 
    Chrome browser. My version of Chrome states it is v54, so I downloaded
    `2.26`.

2. Extract the binary from the archive

3. Move it to a directory that we will place in our PATH. I created the following 
directory and added this to my path

    ```shell
    C:\Program Files\ChromeDriver
    ```

4. Test that chrome is available in the PATH

    ```shell
    > chromedriver --version
    ChromeDriver 2.25.426923 (0390b88869384d6eb0d5d09729679f934aab9eed)
    ```

## (Optional) Install Sublime Text 3

Any editor is fine for use with the class. The instructor will switch between 
`vi` and `Sublime Text`, depending on the work being performed. You may 
also install and use Sablime Text of you wish by either downloading and installing
the binary or using Homebrew -- which will be shown here.

1. [Download the installation binary for your environment](https://www.sublimetext.com/3).

2. Run the installer or download and configure the portable archive of the software

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

    * On Windows I copied them to 
    `%APPDATA%\Sublime Text 3\Packages\Sublime Text 3 Snippets`
    like the documentation stated and it worked correctly. **Note**:
    The `Sublime Text 3` directory is not created for us until we
    launch the editor at least once. I also found I had to manually
    create the last `Sublime Text 3 Snippets` directory in the path.

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
that are unique to Windows. At this point in time we can return to 
the generic instructions to complete our full stack development 
environment setup.
