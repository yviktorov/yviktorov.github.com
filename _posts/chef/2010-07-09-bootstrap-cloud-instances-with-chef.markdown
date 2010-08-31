---
layout: post
title: Bootstrap cloud instances with a Chef and Opscode
categories: [chef]
truncatewords: 45
---

I was looking for cloud automation approaches and come up with some
interesting pragmatic solution.

If shortly, this is about provisioning [Chef Client][] ready server on
[Rackspace Cloud][] services with a single command. We also will be
using [Opscode][] instead of our own [Chef Server][].

**UPDATE(31 Aug 2010):** forgot to mention `server_url` attribute
  which are important for future findings of chef server, i.e. [Opscode][]
  server in our case. See **Bootstrap Rackspace Cloud server with
  chef-client daemon** section.

## Pre-requirements

 * [Rackspace Cloud][] account
 * [Opscode][] account
 * Follow [Opscode quick start guide][] first

## Sharpening Knife for the cloud

I'll assume we continue with [Opscode quick start guide][] for simplicity.

At this point you must have workstation with a basic [Chef Repository][] and [Chef Client][] installed.

So, let's tweak [Knife][] configuration to let it interact with a [Rackspace Cloud][] API. Open `~/chef-repo/.chef/knife.rb` and add the following:

    knife(
      :rackspace_api_key => "Your Rackspace Cloud API key here",
      :rackspace_api_username => "Your Rackspace Cloud username here"
    )

Preparing necessary cookbooks([Chef Cookbook][] with dependencies) and pushing them to the [Opscode][] server:

    knife cookbook site vendor chef --dependencies
    rake upload_cookbooks

Ok, Knife is sharp now, cookbook opened on the right page, let's bootstrap client ;)

## Bootstrap Rackspace Cloud server with chef-client daemon

> Yes, [Chef Client][] will be running as persistent daemon in order
> to "talk" to [Opscode][] server periodically(about 30 minutes).

Create `~/chef-repo/roles/chef4test.rb` file with the following,
replacing **ORGNAME** with your organization's simple string name:

    name "chef4test"
    description "Bootstraping chef node"
    run_list("recipe[chef::bootstrap_client]")
    default_attributes "chef" => { "server_url" => "https://api.opscode.com/organizations/ORGNAME" }

And throw the following commands:

    cd ~/chef-repo
    rake roles
    knife rackspace server create 'role[chef4test]' --server-name chef4test --image 49 --flavor 2

 * image 49 it's Ubuntu 10.04 LTS (Lucid Lynx);
 * flavor it's a server size, i.e. 512MB RAM with 20GB disk for $0.03 per hour in this case.

> And image 4 it's Debian 5.0 (Lenny) if you interested ;)

Now wait a little and check [Opscode Console][] when [Knife][] complete.

> However it's easy to discover the flavor id, I still did not found an easy way to discover image ID and using [FOG][] library(which also required by rackspace server sub-commands of [Knife][] btw).

## Removing servers

    knife rackspace server delete <ID>

You can discover server ID with a list command:

    knife rackspace server list

## Related posts

 * [Getting dynamic on the cloud part #1](http://blog.zefironetworks.com/?p=21)
 * [Using puppet in UEC/EC2: Automating the signing process](http://ubuntumathiaz.wordpress.com/2010/03/25/using-puppet-in-uecec2-automating-the-signing-process/)
 * [Introducing cloud-init's cloud-config syntax](http://ubuntu-smoser.blogspot.com/2010/03/introducing-cloud-inits-cloud-config.html)
 * [Automating EC2 instance setup with user-data scripts](http://www.turnkeylinux.org/blog/ec2-userdata)


[Chef]: http://wiki.opscode.com/display/chef/Home
[Chef Client]: http://wiki.opscode.com/display/chef/Chef+Client
[Chef Server]: http://wiki.opscode.com/display/chef/Chef+Server
[Chef Repository]: http://wiki.opscode.com/display/chef/Chef+Repository
[Chef Cookbook]: http://github.com/opscode/cookbooks/tree/master/chef/
[Knife]: http://wiki.opscode.com/display/chef/Knife
  "command-line utility used to interact with a Chef server directly through the RESTful API"
[Rackspace Cloud]: http://www.rackspacecloud.com/
[Opscode]: http://www.opscode.com/
[Opscode Console]: http://manage.opscode.com/
[Opscode knowledge base]: http://help.opscode.com/faqs
[Opscode quick start guide]: http://help.opscode.com/faqs/start/how-to-get-started
[FOG]: http://github.com/geemus/fog
