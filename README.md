Visage
======

Visage is a web interface for viewing [collectd](http://collectd.org) statistics.

It also provides a [JSON](http://json.org) interface onto `collectd`'s RRD data,
giving you an easy way to mash up the data.

Features
--------

 * renders graphs in the browser, and retrieves data asynchronously
 * interactive graph keys, to highlight lines and toggle line visibility
 * drop-down or mouse selection of timeframes (also rendered asynchronously)
 * JSON interface onto `collectd` RRDs
 * Support for FLUSH using either collectd or rrdcached

Here, have a graph:

![Something I prepared earlier.](http://farm2.static.flickr.com/1020/4730994504_c8c6fc9c18_z.jpg)

Or check out [a live demo](http://visage.unstated.net/nadia/cpu+load).

Installing
----------

N.B: Visage must be deployed on a machine where `collectd` stores its stats in RRD.

### Ubuntu ###

On Ubuntu, to install dependencies run:

``` bash
sudo apt-get install -y build-essential librrd-ruby ruby ruby-dev rubygems collectd
```

Then install the app with:

``` bash
gem install visage-app
```

### CentOS/RHEL ###

#### CentOS/RHEL 5 ####
Visage uses [yajl-ruby](https://github.com/brianmario/yajl-ruby) to work with
JSON, which requires Ruby >= 1.8.6. CentOS/RHEL 5 ship with Ruby 1.8.5, so you
will need to use [Ruby Enterprise Edition](http://www.rubyenterpriseedition.com/).

[Endpoint](http://endpoint.com) provide packages for REE and a [Yum repository](https://packages.endpoint.com/)
to ease installation.

Follow the above instructions for installing REE, and then run:

``` bash
sudo yum install -y librrd-dev ruby rubygems collectd
gem install librrd
```

Then install the app with:

``` bash
gem install visage-app
```

#### CentOS/RHEL 6+ ####

On CentOS 6, to install dependencies run:

``` bash
sudo yum install -y ruby-RRDtool ruby ruby-devel rubygems collectd
```

Then install the app with:

``` bash
gem install visage-app
```

### Mac OS X ###

Visage is not supported on Mac OS X, as RRDtool is a pain in the arse on that
platform. It's highly recommended you use [Vagrant](http://vagrantup.com/) to
fire up an Ubuntu box to run Visage.


Running
-------

You can try out Visage quickly with:

``` bash
visage-app start
```

Then paste the URL from the output into your browser.

If you get a `command not found` when running the above command (RubyGems likely
isn't on your PATH), try this instead:

``` bash
$(dirname $(dirname $(gem which visage-app)))/bin/visage-app start
```

Deploying
---------

Visage can be deployed on Apache with Passenger:

``` bash
sudo apt-get install libapache2-mod-passenger
```

Visage can attempt to generate an Apache vhost config for use with Passenger:

``` bash
visage-app genapache
<VirtualHost *>
  ServerName ubuntu.localdomain
  ServerAdmin root@ubuntu.localdomain

  DocumentRoot /home/user/.gem/ruby/1.8/gems/visage-app-0.1.0/lib/visage-app/public

  <Directory "/home/user/.gem/ruby/1.8/gems/visage-app-0.1.0/lib/visage-app/public">
     Options FollowSymLinks Indexes
     AllowOverride None
     Order allow,deny
     Allow from all
   </Directory>
</VirtualHost>
```

Copypasta this into your system's Apache config structure and tune to taste.

To do this on Debian/Ubuntu:

``` bash
sudo -s
visage-app genapache > /etc/apache2/sites-available/visage
a2ensite visage
a2dissite default
service apache2 reload
```

Then head to your Apache instance and Visage will be up and running.

Configuring
-----------

Visage looks for some environment variables when starting up:

  * `CONFIG_PATH`, an entry on the configuration file search path
  * `RRDDIR`, the location of collectd's RRDs
  * `COLLECTDSOCK`, the location of collectd's Unix socket
  * `RRDCACHEDSOCK`, the location of rrdcached's Unix socket

Visage has a configuration search path which can be used for overriding
individual files. By default it has one entry: `$VISAGE_ROOT/lib/visage/config/`.
You can set the `CONFIG_PATH` environment variable to add another directory to
the config load path. This directory will be searched when loading up
configuration files:

```
CONFIG_PATH=/var/lib/visage visage-app start
```

This is especially useful when you want to deploy + run Visage from an installed
gem with Passenger. e.g.

```
<VirtualHost *:80>
  ServerName monitoring.example.org
  ServerAdmin me@example.org

  SetEnv CONFIG_PATH /var/lib/visage
  SetEnv RRDDIR /opt/collectd/var/lib/collectd

  DocumentRoot /var/lib/gems/1.8/gems/visage-app-0.3.0/lib/visage/public
  <Directory />
    Options FollowSymLinks
    AllowOverride None
  </Directory>

  LogFormat "%h %l %u %t \"%r\" %>s %b" common
  CustomLog /var/log/apache2/access.log common
</VirtualHost>
```

Also to keep in mind when deploying with Passenger, the `CONFIG_PATH` directory
and its files need to have the correct ownership:

``` bash
chown nobody:nogroup -R /var/lib/visage
```

Developing + testing
--------------------

Check out the code with:

``` bash
git clone git://github.com/auxesis/visage.git
```

Install the development dependencies with:

``` bash
gem install bundler
bundle install
```

Run all cucumber features:

``` bash
rake features
```

And run the app with:

``` bash
shotgun lib/visage-app/config.ru
```

To create and install a new gem from the current source tree:

``` bash
rake install
```

Releasing
---------

1. Bump the version in lib/visage-app/version.rb
2. Add an entry to CHANGELOG.md
3. `git commit` everything.
4. Build the gem with `rake build`
5. Push the gem to RubyGems.org with `rake push`

Licencing
---------

Visage is MIT licensed.

Visage is distributed with Highcharts. Torstein Hønsi has kindly granted
permission to distribute Highcharts under the GPLv2 as part of Visage.

If you ever need an excellent JavaScript charting library, please consider
purchasing a [commercial license](http://highcharts.com/license) for
Highcharts.

TODO
----

 - add natural language support for timeframes (default view: 1 week ago)
 - add sparklines to profile view
 - add checkbox to enable live updates on all graphs
