---
layout: missunderstood
title: What is chef?
description: Chef is a systems integration framework, built to bring the benefits of configuration management to your entire infrastructure.
---

In this context the [Chef][] is a systems integration framework, built to bring the benefits of configuration management to your entire infrastructure. With [Chef][], you can:

 * Manage your servers by writing code, not by running commands. (via [Cookbooks][])
 * Integrate tightly with your applications, databases, LDAP directories, and more. (via [Libraries][])
 * Easily configure applications that require knowledge about your entire infrastructure ("What systems are running my application?" "What is the current master database server?")

## Chef's related posts
{% for post in site.categories.chef%}
    {% include posts-list.html %}
{% endfor %}

[Chef]: http://wiki.opscode.com/display/chef/Home
[Cookbooks]: http://wiki.opscode.com/display/chef/Cookbooks
[Libraries]: http://wiki.opscode.com/display/chef/Libraries
