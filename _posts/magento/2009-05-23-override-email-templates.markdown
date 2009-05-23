---
layout: post
title: How to override email templates in Magento
categories: [magento]
---

In this post I would like to share the idea of overriding default email templates in  a **programmatic** way and on per design basis. So, you can copy email templates from `app/locale/en_US/template/email` to `app/design/frontend/<yourInterface>/default/locale/en_US/template/email` and modify them as you like.

**NOTE:** _I'll use naming from some real project, i.e. you most likely will need to change_ `TJU_` _prefix and_ `tjuInterface` _to the name of your design package._ _This has been tested with magento 1.3.1 and might not work(or to be different) for others versions._

## Step 1. Override Translate model

**Translate** model "responsible" for the path to email templates, see `getTemplateFile` method in `app/code/core/Mage/Core/Model/Translate.php`. We will be creating 3 files to extend it's default functionality:

 * `app/code/local/TJU/Core/Model/Translate.php`;
 * `app/code/local/TJU/Core/etc/config.xml`;
 * `app/etc/modules/TJU_All.xml`.

**1.1** Create Translate model class in `app/code/local/TJU/Core/Model/Translate.php`.

    {% highlight php %}
    <?php
    class TJU_Core_Model_Translate extends Mage_Core_Model_Translate
    {

        /**
         * Retrieve translated template file
         * Try current design package first
         *
         * @param string $file
         * @param string $type
         * @param string $localeCode
         * @return string
         */
        public function getTemplateFile($file, $type, $localeCode=null)
        {
            if (is_null($localeCode) || preg_match('/[^a-zA-Z_]/', $localeCode)) {
                $localeCode = $this->getLocale();
            }

            // Retrieve template from the current design package
            $designPackage = Mage::getModel('core/design_package');
            $filePath = Mage::getBaseDir('design')  . DS . 'frontend' . DS
                      . $designPackage->getPackageName() . DS . 'default' . DS . 'locale' . DS
                      . $localeCode . DS . 'template' . DS . $type . DS . $file;

            if (!file_exists($filePath)) { // If no template for current design package, use default workflow
                return parent::getTemplateFile($file, $type, $localeCode);
            }

            $ioAdapter = new Varien_Io_File();
            $ioAdapter->open(array('path' => Mage::getBaseDir('locale')));

            return (string) $ioAdapter->read($filePath);
        }
    }
    {% endhighlight %}

**1.2** Configure magento to use your class instead of default one. Create `app/code/local/TJU/Core/etc/config.xml`.

    {% highlight xml %}
    <?xml version="1.0"?>
    <config>
        <global>
            <models>
                <core>
                    <rewrite>
                      <translate>TJU_Core_Model_Translate</translate>
                    </rewrite>
                </core>
            </models>
        </global>
    </config>
    {% endhighlight %}

**1.3** Enable our module. Create `app/etc/modules/TJU_All.xml`.

    {% highlight xml %}
    <?xml version="1.0"?>
    <config>
         <modules>
            <TJU_Core>
                <active>true</active>
                <codePool>local</codePool>
            </TJU_Core>
         </modules>
    </config>
    {% endhighlight %}

**1.4** Clear cache, just `rm -rf var/*` for now ;)

## Step 2. Modify email templates

Now copy email templates from `app/locale/en_US/template/email` to `app/design/frontend/tjuInterface/default/locale/en_US/template/email` and modify them as you like.

**NOTE:** _I found nice script which helps to override all dummy data at once,_ [take a look](http://www.bearsols.com/how-to-change-the-magento-email-templates-a103.html), _pls. For some reason this did not work out of box for me, at magento 1.3.1, so I made some adjustments, see below._

Create `email.sh` file and make it executable.

    {% highlight sh %}
    #!/bin/sh
    phone='(XXX) XXX-XXXX'
    email='example@example.com'
    basedir='app/design/frontend/tjuInterface/default'

    find $basedir/locale/en_US/template/email/sales -type f -exec sed -i 's/alt="Magento"/alt="{{var order.getStoreGroupName()}}"/g' '{}' \;
    find $basedir/locale/en_US/template/email/sales -type f -exec sed -i 's/Magento Demo Store/{{var order.getStoreGroupName()}} /g' '{}' \;
    find $basedir/locale/en_US/template/email/sales -type f -exec sed -i 's/Demo Store/{{var order.getStoreGroupName()}} /g' '{}' \;
    find $basedir/locale/en_US/template/email/sales -type f -exec sed -i "s/mailto:magento@varien.com/mailto:$email/g" '{}' \;
    find $basedir/locale/en_US/template/email/sales -type f -exec sed -i "s/dummyemail@magentocommerce.com/$email/g" '{}' \;

    find $basedir/locale/en_US/template/email -type f -exec sed -i 's/alt="Magento"/alt="{{var customer.store.name}}"/g' '{}' \;
    find $basedir/locale/en_US/template/email -type f -exec sed -i 's/Magento Demo Store/{{var customer.store.name}} /g' '{}' \;
    find $basedir/locale/en_US/template/email -type f -exec sed -i 's/Demo Store/{{var customer.store.name}} /g' '{}' \;
    find $basedir/locale/en_US/template/email -type f -exec sed -i "s/mailto:magento@varien.com/mailto:$email/g" '{}' \;
    find $basedir/locale/en_US/template/email -type f -exec sed -i "s/dummyemail@magentocommerce.com/$email/g" '{}' \;

    find $basedir/locale/en_US/template/email -type f -exec sed -i "s/(555) 555-0123/$phone/g" '{}' \;
    {% endhighlight %}

Thanks.

## References

 * [Customize Models](http://www.magentocommerce.com/wiki/how-to/customize_part_of_configuration#models)

