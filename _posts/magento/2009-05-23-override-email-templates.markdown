---
layout: post
title: How to override email templates in Magento
categories: [magento]
truncatewords: 40
---

In this post I would like to share the idea of overriding default email templates in  a **programmatic** way and on per design basis. So, you can copy email templates from `app/locale/en_US/template/email` to `app/design/frontend/<yourInterface>/default/locale/en_US/template/email` and modify them as you like.

**NOTE:** _I'll use naming from some real project, i.e. you most likely will need to change_ `TJU_` _prefix and_ `tjuInterface` _to the name of your design package._ _This has been tested with magento 1.3.1 and might not work(or to be different) for others versions._

## Step 1. Override Translate model

**Translate** model "responsible" for the path to email templates, see `getTemplateFile` method in `app/code/core/Mage/Core/Model/Translate.php`. We will be creating 3 files to extend it's default functionality:

 * `app/code/local/TJU/Core/Model/Translate.php`;
 * `app/code/local/TJU/Core/etc/config.xml`;
 * `app/etc/modules/TJU_All.xml`.

**1.1** Create Translate model class in `app/code/local/TJU/Core/Model/Translate.php`.

<script src="http://gist.github.com/369205.js">
</script>

**1.2** Configure magento to use your class instead of default one. Create `app/code/local/TJU/Core/etc/config.xml`.

<script src="http://gist.github.com/369206.js">
</script>

**1.3** Enable our module. Create `app/etc/modules/TJU_All.xml`.

<script src="http://gist.github.com/369208.js">
</script>

**1.4** Clear cache, just `rm -rf var/*` for now ;)

## Step 2. Modify email templates

Now copy email templates from `app/locale/en_US/template/email` to `app/design/frontend/tjuInterface/default/locale/en_US/template/email` and modify them as you like.

I also found nice script which helps to override all dummy data at once, [take a look](http://www.bearsols.com/how-to-change-the-magento-email-templates-a103.html), pls. For some reason it did not work out of box for me, with magento 1.3.1, so I made some adjustments, see below.

Create `email.sh` file and make it executable.

<script src="http://gist.github.com/369209.js">
</script>

Thanks.

## References

 * [Customize Models](http://www.magentocommerce.com/wiki/how-to/customize_part_of_configuration#models)

