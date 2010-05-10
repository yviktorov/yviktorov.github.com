---
layout: post
title: Distribute Magento extensions as a PEAR packages
categories: [magento]
truncatewords: 50
---

Many users and developers do not know that Magento uses a [PEAR][]-repository for installation and management of its extensions. Some of you might now ask: What is [PEAR][]? [PEAR][] (PHP Extension and Application Repository) is - as the name indicates already - a repository for PHP-extensions and applications. [PEAR][] was set up in 2001 and is managed by the [PEAR group][], which was founded for this purpose. This repository currently offers more than 550 so-called packages with software provided under the [PHP-license][] for download. Most of them are components, thus software that is not able to run on its own, which can be integrated in another software application.

Administration of the packages is done using a so-called [PEAR channel][], allowing the developer to download the package directly. Nevertheless, it is much more comfortable to install the packages using a local PEAR repository which is included in every current Linux-distribution and every Apple-computer.

In this post I would like to cover some practices of distributing free and paid magento extensions though [PEAR channel][].

## Magento Connect

[Magento Connect][] it's not only the official marketplace for magento extensions, but also [PEAR channel][] for distribution of core and community extensions which can be installed in the one of the following ways:

 * [PEAR command line][]
 * [Magento Connect Manager][]

Both methods are compatible, the [Magento Connect Manager][] it's a web interface to [PEAR command line][]. You can also install extensions manually, but that's not effective and might lead to serious maintenance problems in future.

If you developing Open Source extensions this is the best way to go, after you get familiar with [packaging a magento extension][] continue with "[Creating a Magento Community extension][]" :)

## Simple personal PEAR channel

There are three more or less known implementations we can use to establish own [PEAR channel][]:

 * [PEAR2_SimpleChannelServer](http://pear.php.net/manual/en/channels.scs.php)
 * [Chiara_PEAR_Server](http://greg.chiaraquartet.net/archives/123-Setting-up-your-own-PEAR-channel-with-Chiara_PEAR_Server-the-official-way.html)
 * [Pirum](http://www.pirum-project.org/)

Every solution has it's advantages/disadvantages, you have to read more about it's features. Here I would like to describe one use case for setting up private(password protected) channel and in order to keep things simple and clean I'll be using Pirum, because: *Pirum consists of just one file, a command line tool, written in PHP. There is no external dependencies, no not need for a database, no need to setup credentials, and nothing need to be installed or configured.*

### Private channel using Subversion

*A PEAR channel server managed by Pirum is simply a directory full of static XML files. As such, the server can be hosted on any shared host, without the need for PHP or any other web server technology installed.* i.e. on any http/https accessible server and it's very popular setup when subversion served by webserver with the help of `WebDAV` module.

*PEAR compatible channels can be password protected. Password protection is done via a HTTP Basic Authentication (.htaccess and .htpassword on Apache).* i.e. this is how password protection works for subversion when it served by webserver.

When I realize this I was felt lucky that in addition to multiple git repositories on per project basis the [codebasehq][] allow to create subversion repositories the same way. So, in order to proceed you must have password protected subversion repository accessible though http/https protocol, but don't worry, I mentioned [codebasehq][] because they also have Free Account you could use for this little experiment ;)

#### Intstall the Pirum and create your channel

As described at Pirum [website](http://www.pirum-project.org/), but with the following adjustments in the `pirum.xml` file, prior to build command:

 * use "tinycarts" for alias (it's to simplify example, you can use your own alias after you get familiar with the concept)
 * use `tinycarts.github.com` for the name (for same as above reason)
 * url must pointing to the repository, i.e. http://mysite.svn.codebasehq.com/foobar/pear/

#### Releasing package

During channel creation step we used "tinycarts" as alias and name for our channel, this is to make it possible to use the following package in this experiment: [Lib_Tinycarts_Debug](http://tinycarts.github.com//get/Lib_Tinycarts_Debug-1.0.0.tgz)

Simply download and execute the following command:

     php pirum add pear /path/to/saved/Lib_Tinycarts_Debug-1.0.0.tgz

#### Import channel into repository

     cd pear
     svn import . http://mysite.svn.codebasehq.com/foobar/pear

**NOTE** if you also considered to go with [codebasehq][] for this experiment:

 * mysite - the name of site you choose when sign up for account
 * foobar - the project short name
 * pear - subversion repository short name

#### Using your PEAR Channel Server

Here is minor problem, you need to associate the login/password with a channel, but you can't discover the channel because it's already password protected :)
So, you need to do some routine, i.e. discover the channel using the channel.xml file manually, execute the following command in magento directory:

    ./pear channel-add /path/to/saved/channel.xml

Now the PEAR manager needs to know about the channel's username and password. We tell him by using set-config in connection with the channel option:

    ./pear config-set -c tinycarts username johndoe
    ./pear config-set -c tinycarts password secret

And finally install the `Lib_Tinycarts_Debug` package:

    ./pear install tinycarts/Lib_Tinycarts_Debug

**NOTE:** if after you set the login/password you still can't access the package, try to clear the cache with the following command:

    ./pear clear-cache

I found it simple and useful for ongoing developments for particular client.

## faett.net tools

As described in [press release][]:

*Companies offering extensions for Magento can now distribute their products as PEAR packages in their shops. This is much easier than the somehow complicated solution using zip-files and ftp-upload. The components provided by faett.net allow*

This is last minute found and I had not chance to try it myself, but it looks very promising, see the following posts for the more info:

 * [faett.net website](http://www.faett.net/?___store=english)
 * [Faett_Manager](http://www.faett.net/blog/cat/faett-manager-en/)
 * [Faett_Channel](http://www.faett.net/blog/cat/faett-channel-en/)

This way seems ideal for distributing paid extensions and will lead in better connection with your customers. And even more simplify their life when they host magento at the servers with no SSH, like [cloudsites][] by [Rackspace Cloud][].

[press release]: http://www.prlog.org/10619989-magento-connect-20-comes-with-faettnet-it-new-media-software.html
[PEAR]: http://pear.php.net/
[PEAR group]: http://pear.php.net/group/
[PEAR channel]: http://pear.php.net/manual/en/channels.whatarethey.php
[php-license]: http://www.php.net/license/
[REST Interface compliant channel server]: http://pear.php.net/manual/en/core.rest.php
[packaging a magento extension]: http://www.magentocommerce.com/wiki/packaging_a_magento_extension
[Magento Connect]: http://www.magentocommerce.com/magento-connect
[PEAR command line]: http://www.magentocommerce.com/wiki/groups/227/installing_magento_via_shell_ssh#use_pear_to_download_and_install_magento_connect_extension
[Magento Connect Manager]: http://www.magentocommerce.com/wiki/how-to/how_tu_use_magento_connect_manager_-_downloader
[Creating a Magento Community extension]: http://www.magentocommerce.com/wiki/how_to_use_magentoconnect#creating_a_magento_community_extension
[codebasehq]: http://www.codebasehq.com/
  "Git hosting, Mercurial hosting & Subversion hosting with complete project management"
[cloudsites]: http://www.rackspacecloud.com/cloud_hosting_products/sites
[Rackspace Cloud]: http://www.rackspacecloud.com/
