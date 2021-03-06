---
title: Configuring Nginx and Phusion Passenger on Debian Lenny
enki_id: 4
tags: [debian, rails, ruby, phusion-passenger]
---
[Continuing on from my previous post on compiling Nginx with Phusion Passenger]({% post_url 2010-09-10-compiling-nginx-and-phusion-passenger-on-debian-lenny %}), some configuration is now in order.

To give credit where due, I'd like to mention that I found much help on the [Slicehost articles and tutorials site](http://articles.slicehost.com/) and adapted some of their instructions for parts of this post. Thanks Slicehost!<!--more-->

I'm using the root account, so there'll be no 'sudo' command used herein. Adjust your commands accordingly if you don't do things this way.

### Adding an Nginx init script

As I compiled Nginx from source, this meant that no init script was created as it would have been were I to install Nginx via a Debian package. So one needs to be created.

If you have Nginx running, stop it:

```bash
# kill `cat /usr/local/nginx/logs/nginx.pid`
```

Create the file that will become the init script:

```bash
# nano /etc/init.d/nginx
```

Paste this into /etc/init.d/nginx:

```bash
#! /bin/sh

### BEGIN INIT INFO
# Provides:          nginx
# Required-Start:    $all
# Required-Stop:     $all
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the nginx web server
# Description:       starts nginx using start-stop-daemon
### END INIT INFO

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/local/sbin/nginx
NAME=nginx
DESC=nginx

test -x $DAEMON || exit 0

# Include nginx defaults if available
if [ -f /etc/default/nginx ] ; then
        . /etc/default/nginx
fi

set -e

case "$1" in
  start)
        echo -n "Starting $DESC: "
        start-stop-daemon --start --quiet --pidfile /usr/local/nginx/logs/$NAME.pid \
                --exec $DAEMON -- $DAEMON_OPTS
        echo "$NAME."
        ;;
  stop)
        echo -n "Stopping $DESC: "
        start-stop-daemon --stop --quiet --pidfile /usr/local/nginx/logs/$NAME.pid \
                --exec $DAEMON
        echo "$NAME."
        ;;

  restart|force-reload)
        echo -n "Restarting $DESC: "
        start-stop-daemon --stop --quiet --pidfile \
                /usr/local/nginx/logs/$NAME.pid --exec $DAEMON
        sleep 1
        start-stop-daemon --start --quiet --pidfile \
                /usr/local/nginx/logs/$NAME.pid --exec $DAEMON -- $DAEMON_OPTS
        echo "$NAME."
        ;;
  reload)
      echo -n "Reloading $DESC configuration: "
      start-stop-daemon --stop --signal HUP --quiet --pidfile /usr/local/nginx/logs/$NAME.pid \
          --exec $DAEMON
      echo "$NAME."
      ;;
  *)
        N=/etc/init.d/$NAME
        echo "Usage: $N {start|stop|restart|reload|force-reload}" >&2
        exit 1
        ;;
esac

exit 0
```

Make the file executable:

```bash
# chmod +x /etc/init.d/nginx
```

Add the script to the default run levels:

```bash
# /usr/sbin/update-rc.d -f nginx defaults
```

Now you should be able to start, stop and restart Nginx just as with any other service:

```bash
# /etc/init.d/nginx start
...
# /etc/init.d/nginx stop
...
# /etc/init.d/nginx restart
```

Make sure you actually use the full path as above, rather than typing something like:

```bash
# nginx restart
```

Because if you type just 'nginx restart' from some random directory on your server, you'll actually be calling the Nginx binary rather than the Nginx init script and things will not work as expected.

The init script will also be called on a reboot, so Nginx will automatically start.

### Mirroring the standard Debian layout for Nginx

If you've used Apache on Debian, you'll probably remember that it makes use of particular conventions regarding its files and directories. Most notably - and I think this is Debian specific - it makes use of two directories: 'sites-available' and 'sites-enabled'. The 'sites-available' directory contains the configuration files for all virtual hosts and you then symlink specific configuration files from 'sites-available' to the 'sites-enabled' directory in order for those sites to be, well, enabled. If I had installed Nginx via a Debian package, this is the layout I would have ended up with and I like these conventions, so this is what I'm going to mirror for my compiled form source Nginx.

Create the two main folders:

```bash
# mkdir /usr/local/nginx/sites-available
...
# mkdir /usr/local/nginx/sites-enabled
```

#### Edit the default config file

Create a backup of the default Nginx config file (just in case):

```bash
# cp /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf.bak
```

Now edit the original:

```bash
nano /usr/local/nginx/conf/nginx.conf
```

Delete the contents of nginx.conf and paste this in its place:

```bash
user www-data;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    gzip  on;

    passenger_root /usr/local/lib/ruby/gems/1.9.1/gems/passenger-2.2.15;

    include /usr/local/nginx/sites-enabled/*;

}
```

Save and close the file. Note the line 'include /usr/local/nginx/sites-enabled/*;' this is what tells Nginx to look in our '/usr/local/nginx/sites-enabled/' directory for extra configuration files.

#### Create a default virtual host

We're next going to create a default virtual host config file:

```bash
# nano /usr/local/nginx/sites-available/default
```

Paste this into the file:

```bash
server {
    listen       80;
    server_name  localhost;

    location / {
        root   html;
        index  index.html index.htm;
    }


    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }

}
```

#### Create the symlink

So Nginx only looks in '/usr/local/nginx/sites-enabled/' for extra configuration files and our new default virtual host config file is in '/usr/local/nginx/sites-available/', this means we need to now create a symlink from one to the other:

```bash
# ln -s /usr/local/nginx/sites-available/default /usr/local/nginx/sites-enabled/default
```

Restart Nginx and browse to whatever URL is the default for your server and you should get the standard "Welcome to nginx!' page. If you ever want to disable this default virtual host, just delete the symlink and restart Nginx.

### Finishing touches to Rails support

So the above should get you going OK for serving up basic pages with Nginx using virtual hosts. But I'm actually wanting to run Rails apps on top of Nginx. When I compiled Nginx I added in the Phusion Passenger module so now I just have to enable it with the virtual host for a Rails app.

Let's assume that DNS is all set up and you want to create an Nginx virtual host to run your new Rails app at http://fizzbuzz.com/.

Create the location that will store your website files:

```bash
# mkdir /var/www/fizzbuzz.com
```

Put your Rails app's files into this new directory. Note that you really can create this directory anywhere and call it whatever you like. I'm just once again roughly following Debian conventions and my own preference here.

Create the virtual host config file:

```bash
# nano /usr/local/nginx/sites-available/fizzbuzz.com
```

Paste this into the file:

```bash
server {

            listen   80;
            server_name  www.fizzbuzz.com;
            rewrite ^/(.*) http://fizzbuzz.com/$1 permanent;

           }

server {

            listen   80;
            server_name fizzbuzz.com;
            root /var/www/fizzbuzz.com/public;
            passenger_enabled on;

            access_log /var/www/fizzbuzz.com/log/access.log;
            error_log /var/www/fizzbuzz.com/log/error.log;

            }
```

Some things to note are that the first server block just rewrites 'www.fizzbuzz.com' to 'fizzbuzz.com', if you have a desperate need for the 'www' then you can switch this around and change the 'server_name' in the second server block accordingly.

The second server block sets the public root of the Rails app, enables the Phusion Passenger module for this virtual host and sets the location for a couple of log files. You may also have to manually create these log files to start off with:

```bash
# touch /var/www/fizzbuzz.com/log/access.log
...
# touch /var/www/fizzbuzz.com/log/error.log
```

Create the symlink:

```bash
# ln -s /usr/local/nginx/sites-available/fizzbuzz.com /usr/local/nginx/sites-enabled/fizzbuzz.com
```

Restart Nginx and you should be all set to ride Nginx down the rails.

I haven't mentioned the use of anything like [Capistrano](http://www.capify.org) for deployment. I may get around to writing about Capistrano at some point, but it's not like I really need to, there's heaps that's been written on it already and I'm hardly an expert. If you're looking for a more automated way to deploy a Rails app that's probably a bit safer (at least for more complex projects) than just uploading to a directory and restarting the web server, then I encourage you to check out Capistrano. Sometimes though, uploading files and restarting the web server is exactly the straight forward process that I want and just using Nginx plus Phusion Passenger makes this easy.
