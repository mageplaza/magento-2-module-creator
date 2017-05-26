# How to create simple shipping method in Magento 2



## Step 1: Create config.xml file


Create a module call: `Simpleshipping` located in `app/code/Mageplaza/Simpleshipping`

![shipping method in magento 2](https://i.imgur.com/xNM7TOu.png)

- [app/code/Mageplaza/Simpleshipping/registration.php](https://github.com/mageplaza/magento-2-shipping-method/blob/master/registration.php)
- [app/code/Mageplaza/Simpleshipping/etc/module.xml](https://github.com/mageplaza/magento-2-shipping-method/blob/master/etc/module.xml)

Now create `config.xml` at `app/code/Mageplaza/Simpleshipping/etc/config.xml`

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Store:etc/config.xsd">
    <default>
        <carriers>
            <simpleshipping>
                <enable>1</enable>
                <sallowspecific>0</sallowspecific>
                <model>Mageplaza\Simpleshipping\Model\Carrier\Shipping</model>
                <name>Mageplaza Sample Shipping Method</name>
                <price>10.00</price>
                <title>Mageplaza Sample Shipping Method</title>
                <type>I</type>
                <specific_error_msg>This shipping method is not available. To use this shipping method, please contact us.</specific_error_msg>
            </simpleshipping>
        </carriers>
    </default>
</config>
```

The shipping method code should be child of `default > carriers`. And this code should be exactly same with $_code in `app/code/Mageplaza/Simpleshipping/Model/Carrier/Shipping.php:13`

## Step 2: Create shipping model

Next, we need to create a model class for this **shipping method in Magento 2**

Create a `Shipping.php` at `app/code/Mageplaza/Simpleshipping/Model/Carrier/Shipping.php`

``` php
<?php
namespace Mageplaza\Simpleshipping\Model\Carrier;

use Magento\Quote\Model\Quote\Address\RateRequest;
use Magento\Shipping\Model\Rate\Result;

class Shipping extends \Magento\Shipping\Model\Carrier\AbstractCarrier implements
    \Magento\Shipping\Model\Carrier\CarrierInterface
{
    /**
     * @var string
     */
    protected $_code = 'simpleshipping';

    /**
     * Shipping constructor.
     *
     * @param \Magento\Framework\App\Config\ScopeConfigInterface          $scopeConfig
     * @param \Magento\Quote\Model\Quote\Address\RateResult\ErrorFactory  $rateErrorFactory
     * @param \Psr\Log\LoggerInterface                                    $logger
     * @param \Magento\Shipping\Model\Rate\ResultFactory                  $rateResultFactory
     * @param \Magento\Quote\Model\Quote\Address\RateResult\MethodFactory $rateMethodFactory
     * @param array                                                       $data
     */
    public function __construct(
        \Magento\Framework\App\Config\ScopeConfigInterface $scopeConfig,
        \Magento\Quote\Model\Quote\Address\RateResult\ErrorFactory $rateErrorFactory,
        \Psr\Log\LoggerInterface $logger,
        \Magento\Shipping\Model\Rate\ResultFactory $rateResultFactory,
        \Magento\Quote\Model\Quote\Address\RateResult\MethodFactory $rateMethodFactory,
        array $data = []
    ) {
        $this->_rateResultFactory = $rateResultFactory;
        $this->_rateMethodFactory = $rateMethodFactory;
        parent::__construct($scopeConfig, $rateErrorFactory, $logger, $data);
    }

    /**
     * get allowed methods
     * @return array
     */
    public function getAllowedMethods()
    {
        return [$this->_code => $this->getConfigData('name')];
    }

    /**
     * @param RateRequest $request
     * @return bool|Result
     */
    public function collectRates(RateRequest $request)
    {
        if (!$this->getConfigFlag('enable')) {
            return false;
        }

        /** @var \Magento\Shipping\Model\Rate\Result $result */
        $result = $this->_rateResultFactory->create();

        /** @var \Magento\Quote\Model\Quote\Address\RateResult\Method $method */
        $method = $this->_rateMethodFactory->create();

        $method->setCarrier($this->_code);
        $method->setCarrierTitle($this->getConfigData('title'));

        $method->setMethod($this->_code);
        $method->setMethodTitle($this->getConfigData('name'));

        $amount = $this->getConfigData('price');

        $method->setPrice($amount);
        $method->setCost($amount);

        $result->append($method);

        return $result;
    }
}
```

## Step 3: Create configuration file

Now create `system.xml` file at `app/code/Mageplaza/Simpleshipping/etc/adminhtml/system.xml`

Learn more [System.xml configuration for Magento 2](https://www.mageplaza.com/magento-2-module-development/create-system-xml-configuration-magento-2.html)

with the following content:

``` xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Config:etc/system_file.xsd">
    <system>
        <section id="carriers" translate="label" type="text" sortOrder="320" showInDefault="1" showInWebsite="1"
                 showInStore="1">
            <group id="simpleshipping" translate="label" type="text" sortOrder="0" showInDefault="1" showInWebsite="1"
                   showInStore="1">
                <label>Mageplaza Simple Shipping Method</label>
                <field id="enable" translate="label" type="select" sortOrder="1" showInDefault="1" showInWebsite="1"
                       showInStore="0">
                    <label>Enabled</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="name" translate="label" type="text" sortOrder="3" showInDefault="1" showInWebsite="1"
                       showInStore="1">
                    <label>Method Name</label>
                </field>
                <field id="price" translate="label" type="text" sortOrder="5" showInDefault="1" showInWebsite="1"
                       showInStore="0">
                    <label>Price</label>
                    <validate>validate-number validate-zero-or-greater</validate>
                </field>
                <field id="handling_type" translate="label" type="select" sortOrder="7" showInDefault="1"
                       showInWebsite="1" showInStore="0">
                    <label>Calculate Handling Fee</label>
                    <source_model>Magento\Shipping\Model\Source\HandlingType</source_model>
                </field>
                <field id="handling_fee" translate="label" type="text" sortOrder="8" showInDefault="1" showInWebsite="1"
                       showInStore="0">
                    <label>Handling Fee</label>
                    <validate>validate-number validate-zero-or-greater</validate>
                </field>
                <field id="sort_order" translate="label" type="text" sortOrder="100" showInDefault="1" showInWebsite="1"
                       showInStore="0">
                    <label>Sort Order</label>
                </field>
                <field id="title" translate="label" type="text" sortOrder="2" showInDefault="1" showInWebsite="1"
                       showInStore="1">
                    <label>Title</label>
                </field>
                <field id="sallowspecific" translate="label" type="select" sortOrder="90" showInDefault="1"
                       showInWebsite="1" showInStore="0">
                    <label>Ship to Applicable Countries</label>
                    <frontend_class>shipping-applicable-country</frontend_class>
                    <source_model>Magento\Shipping\Model\Config\Source\Allspecificcountries</source_model>
                </field>
                <field id="specific_country" translate="label" type="multiselect" sortOrder="91" showInDefault="1"
                       showInWebsite="1" showInStore="0">
                    <label>Ship to Specific Countries</label>
                    <source_model>Magento\Directory\Model\Config\Source\Country</source_model>
                    <can_be_empty>1</can_be_empty>
                </field>
                <field id="show_method" translate="label" type="select" sortOrder="92" showInDefault="1"
                       showInWebsite="1" showInStore="0">
                    <label>Show Method if Not Applicable</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="specific_error_msg" translate="label" type="textarea" sortOrder="80" showInDefault="1"
                       showInWebsite="1" showInStore="1">
                    <label>Displayed Error Message</label>
                </field>
            </group>
        </section>
    </system>
</config>
```


## Step 4: Enable module

This is final step, you need to run upgrade command to enable Simple Shipping method.

```
php bin/magento setup:upgrade
```

[Flush Magento cache](https://www.mageplaza.com/kb/how-flush-enable-disable-cache.html) if any.

