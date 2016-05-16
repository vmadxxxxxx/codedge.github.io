---
layout: post
title: Display subcategories on category pages in Magento
categories: [Magento-1.x, PHP, Web Development]
comments: true
tags: [magento, categories, sub-categories]
slug: 'magento-show-subcategories-on-category-page'
excerpt: Display subcategories on category pages configurable per category in Magento Admin panel via a custom view mode.
---

You might find yourself in the position to create a new display mode for categories in the Magento Admin panel. Based on this new mode you want to display certain elements on the side or do other nifty things.

I am talking about this dropdown, that you reach via `"Categories" > "Manage Categories" > Select a category > Tab "Display Settings" > Dropdown "Display Mode"`. Here we want to add a new entry _Subcategories only_ that is going to make it possible to only show subcategories of a certain category.

![Custom view mode for categories in Magento]({{ site.BASE_PATH }}/assets/media/2016-04-27/magento-admin-category-display-mode.png)

The best way to do it is to write your own module as we need to overwrite some core functionality.

### Create your own module skeleton
Let's first create our basic module structure. The following file and folder structure need to be placed either in `app/code/community` or in `app/code/local`. My further explanations use the _local_ code pool, please keep that in mind.

{% highlight bash %}
├── Block
│   └── Category
│       └── View.php
├── Helper
│   └── Data.php
├── Model
│   └── Catalog
│       ├── Category
│       │   └── Attribute
│       │       └── Source
│       │           └── Mode.php
│       └── Category.php
└── etc
    └── config.xml
{% endhighlight %}

### New entry for the dropdown

#### Prepare the new entry
What we first need is a new constant in class `Mage_Catalog_Model_Category`. As you can see there each already existing entry has a constant value, f. ex. if you select _Products only_ in the dropdown the constant `DM_PRODUCT` with the value _PRODUCTS_ is referenced. 

{% highlight php %}
<?php
class Mage_Catalog_Model_Category extends Mage_Catalog_Model_Abstract
{
    ...

    /**
     * Category display modes
     */
    const DM_PRODUCT            = 'PRODUCTS';             // Display only products on the category page
    const DM_PAGE               = 'PAGE';                 // Display only static blocks on the category page
    const DM_MIXED              = 'PRODUCTS_AND_PAGE';    // Display mixed content, static blocks and products

    ...
{% endhighlight %}

For our new entry we also need a constant value. We will name the constant `DM_SUBCATEGORY` with the value `SUBCATEGORY`. As we do not edit core files, we create our own class that extends the core class and add our new value there.

Create a file `app/code/local/Codedge/Catalog/Model/Catalog/Category.php` with the following content:

{% highlight php %}
<?php
class Codedge_Catalog_Model_Catalog_Category extends Mage_Catalog_Model_Category
{
    /**
     * New display mode for showing subcategories
     */
    const DM_SUBCATEGORY = 'SUBCATEGORY';
}
{% endhighlight %}

That constant will be referenced in the file below.

#### Add a new source entry for the dropdown
What we now actually need is a the new item in the dropdown field. For adding a new entry in the dropdown we need to rewrite the method `getAllOptions()` in class `Mage_Catalog_Model_Category_Attribute_Source_Mode`.
So we easily create our own class and place that file in our module structure at  `app/code/local/Codedge/Catalog/Model/Catalog/Category/Attribute/Source/Mode.php`

{% highlight php %}
<?php
class Codedge_Catalog_Model_Catalog_Category_Attribute_Source_Mode
    extends Mage_Catalog_Model_Category_Attribute_Source_Mode
{
    /**
     * Set a new display mode for categories
     */
    public function getAllOptions()
    {
        $options = parent::getAllOptions();

        $options[] = array(
                'value' => Codedge_Catalog_Model_Catalog_Category::DM_SUBCATEGORY, // here reference our value from above
                'label' => Mage::helper('codedge_catalog')->__('Subcategories only'), // our own helper for translations
        );

        $this->_options = $options;

        return $this->_options;
    }
}
{% endhighlight %}

All we're doing here is to retrieve the original dropdown options and adding a new array with our new entry to it. With the new entry added we just return the array.

##### _Optional step:_ Translation  

As you can see we use our translations and therefore need our own helper. The helper class looks pretty straightforward / empty ;-)
Don't forget to add a translation file in `app/locale/<the locale you want>/` and put in the correct translation. If you don't need any translation you can skip this and use the English term we already put in.

{% highlight php %}
<?php
class Codedge_Catalog_Helper_Data extends Mage_Core_Helper_Abstract
{

}
{% endhighlight %}

#### Add the class rewrites to module configuration

After we created our classes we need to add the proper configuration to the modules config.xml to make the module actually work. As you can see below we rewrite the two modules from above, set up our helper for translations (optional, only if needed), and furthermore create a new block class. The creation of the block class that will help us taking care of outputting the subcategories when selected is covered next.

{% highlight xml %}
<?xml version="1.0"?>
<config>
    <modules>
        <Codedge_Catalog>
            <version>0.1.1</version>
        </Codedge_Catalog>
    </modules>
    <global>
        <models>
            <codedge_catalog>
                <class>Codedge_Catalog_Model</class>
                <resourceModel>codedge_catalog_resource</resourceModel>
            </codedge_catalog>
            <codedge_catalog_resource>
                <class>Codedge_Catalog_Model_Resource</class>
            </codedge_catalog_resource>
            <catalog>
                <rewrite>
                    <category>Codedge_Catalog_Model_Catalog_Category</category>
                    <category_attribute_source_mode>Codedge_Catalog_Model_Catalog_Category_Attribute_Source_Mode</category_attribute_source_mode>
                </rewrite>
            </catalog>
        </models>
        <!-- Optional, only if needed for translation -->
        <helpers>
            <codedge_catalog>
                <class>Codedge_Catalog_Helper</class>
            </codedge_catalog>
        </helpers>
        <!-- End optional -->
        <blocks>
            <codedge_catalog>
                <class>Codedge_Catalog_Block</class>
            </codedge_catalog>
            <catalog>
                <rewrite>
                    <category_view>Codedge_Catalog_Block_Category_View</category_view>
                </rewrite>
            </catalog>
        </blocks>
    </global>
    <!-- Optional, only if needed for translation -->
    <adminhtml>
        <translate>
            <modules>
                <codedge_catalog>
                    <files>
                        <default>Codedge_Catalog.csv</default>
                    </files>
                </codedge_catalog>
            </modules>
        </translate>
    </adminhtml>
    <!-- End optional -->
    <frontend>
        <layout>
            <updates>
                <codedge_catalog>
                    <file>codedge_catalog.xml</file>
                </codedge_catalog>
            </updates>
        </layout>
    </frontend>
</config>
{% endhighlight %}

Finally we got our new entry _Subcategories only_ in the dropdown. The next thing is to actually use the new value to display our subcategories. To use that we have a look into the class `Mage_Catalog_Block_Category_View`. 

{% highlight php %}
<?php
class Mage_Catalog_Block_Category_View extends Mage_Core_Block_Template
{
    ...

    public function getProductListHtml()
    {
        return $this->getChildHtml('product_list');
    }

    ...

    /**
     * Check if category display mode is "Products Only"
     * @return bool
     */
    public function isProductMode()
    {
        return $this->getCurrentCategory()->getDisplayMode()==Mage_Catalog_Model_Category::DM_PRODUCT;
    }

    /**
     * Check if category display mode is "Static Block and Products"
     * @return bool
     */
    public function isMixedMode()
    {
        return $this->getCurrentCategory()->getDisplayMode()==Mage_Catalog_Model_Category::DM_MIXED;
    }

    ...
{% endhighlight %}

We can spot methods, that check what display mode is selected in our dropdown, e. g. `isProductMode()` and we also see a method that returns a child html block, `getProductListHtml()`. So what we need now are two new methods:  

1. The first one for checking if our new entry _Subcategories only_ is selected
2. The second one for returning a new block (incl. template) that will output the subcategories

Let's freshly start with a rewrite of `Mage_Catalog_Block_Category_View` with our own class in the file `app/code/local/Codedge/Catalog/Block/Category/View.php`:

{% highlight php %}
<?php
class Codedge_Catalog_Block_Category_View extends Mage_Catalog_Block_Category_View
{
    /**
     * Check if category display mode is "Subcategories only"
     *
     * @return bool
     */
    public function isSubcategoryMode()
    {
        return $this->getCurrentCategory()->getDisplayMode()==Codedge_Catalog_Model_Catalog_Category::DM_SUBCATEGORY;
    }

    /**
     * @return string
     */
    public function getSubcategoryHtml()
    {
        return $this->getChildHtml('category.subcategories');
    }
}
{% endhighlight %}

What's going on in those methods:

* `isSubcategoryMode` checks whether the display mode for the current category is equal to our subcategory constant, meaning if _Subcategories only_ is selected in the dropdown
* `getSubcategoryHtml` returns a child block with the name _category.subcategories_. We are going to create this block right now.

### The layout update and template
Next thing is to create our new layout block _category.subcategories_ that you already saw in the config.xml and in the `getSubcategoryHtml()` method above. For that we use layout updates, namely our own file `app/design/frontend/base/default/layout/codedge_catalog.xml`.

{% highlight xml %}
<?xml version="1.0"?>
<layout version="1.0.0">
    <catalog_category_default>
        <reference name="category.products">
            <block type="codedge_catalog/category_view" name="category.subcategories"
                   template="catalog/category/view/subcategories.phtml" />
        </reference>
    </catalog_category_default>
</layout>
{% endhighlight %}

Here we are saying that on each category page (layout handle `<catalog_category_default>`) we make a reference to the already existing block _category.products_ and inject a new block with the name _category.subcategories_ - the one that we fetch with our `getSubcategoryHtml()` method above. This new block uses our block class `Codedge_Catalog_Block_Category_View` and a template at `catalog/category/view/subcategories.phtml` that we create now.

The template can be placed either:

* currently used layout: `app/design/frontend/<YOURPACKAGENAME>/<YOURLAYOUTNAME>/template/catalog/category/view/subcategories.phtml`
* base package: `app/design/frontend/base/default/template/catalog/category/view/subcategories.phtml`

{% highlight html %}
<?php
/** @var Codedge_Catalog_Block_Category_View $this */
/** @var Mage_Catalog_Model_Resource_Category_Collection $_categoriesCollection */
$category = $this->getCurrentCategory();
$_categoriesCollection = $category->getCollection()
    ->addAttributeToSelect(array('name','thumbnail'))
    ->addAttributeToFilter('is_active', 1)
    ->addIdFilter($category->getChildren())
    ->addOrderField('position');
?>

<?php $_collectionSize = $_categoriesCollection->count() ?>
<?php $_columnCount = 3; ?>
<?php $i=0; foreach ($_categoriesCollection as $_category): /** @var Codedge_Catalog_Model_Catalog_Category $_category */?>
    <?php if ($i++%$_columnCount==0): ?>
        <ul class="categories-grid">
    <?php endif ?>
    <li class="item<?php if(($i-1)%$_columnCount==0): ?> first<?php elseif($i%$_columnCount==0): ?> last<?php endif; ?>">
        <a href="<?php echo $_category->getUrl(); ?>" title="" class="category-image">
            <img src="<?php echo $_category->getResizedThumbnail(230,230); ?>" width="230" height="230" alt="" />
        </a>
        <h2 class="category-name">
            <a href="<?php echo $_category->getUrl() ?>" title="<?php echo $this->stripTags($_category->getName(), null, true) ?>">
                <?php echo $_category->getName(); ?>
            </a>
        </h2>
    </li>
    <?php if ($i%$_columnCount==0 || $i==$_collectionSize): ?>
        </ul>
    <?php endif ?>
<?php endforeach ?>
<script type="text/javascript">decorateGeneric($$('ul.categories-grid'), ['odd','even','first','last'])</script>
{% endhighlight %}

I decided to go for a grid with 3 columns side by side. Of course you can modify your styling as you like. I added some new classes to the items and the grid view.

You may have spotted it: I also use a custom function for getting a nicely resized category thumbnail. The method `getResizedThumnails()` can be found in the model class `Codedge_Catalog_Model_Catalog_Category`.

{% highlight php startinline %}
<?php
/**
 * Creates a resized image of the categories thumbnail image
 *
 * @param int $width
 * @param int $height
 * @param int $quality
 *
 * @return bool|string
 */
public function getResizedThumbnail($width=130, $height=130, $quality=100)
{

    if (! $this->getThumbnail()) return false;

    $imageUrl = Mage::getBaseDir ( 'media' ) .DS .  "catalog" . DS . "category" . DS . $this->getThumbnail();
    if (! is_file ( $imageUrl )) return false;

    $imageResized = Mage::getBaseDir ( 'media' ) .DS . "catalog" . DS . "category" . DS . "cache" . DS . "cat_resized" . DS . $this->getThumbnail();

    if (! file_exists ( $imageResized ) && file_exists ( $imageUrl ) || file_exists($imageUrl) && filemtime($imageUrl) > filemtime($imageResized)) :

        $imageObj = new Varien_Image($imageUrl);
        $imageObj->constrainOnly ( true );
        $imageObj->keepAspectRatio ( true );
        $imageObj->keepFrame( false );
        $imageObj->keepTransparency( true );
        $imageObj->quality( $quality );
        $imageObj->resize( $width, $height );
        $imageObj->save( $imageResized );
    endif;

    if(file_exists($imageResized)){
        return Mage::getBaseUrl( 'media' ) . "catalog" . DS . "category" . DS . "cache" . DS . "cat_resized" . DS . $this->getThumbnail();
    }else{
        return $this->getThumbnail();
    }
}
{% endhighlight %}

#### Important last step
To finally see our results we need to output or block and/or the template. We copy the file `app/design/frontend/base/default/template/catalog/category/view.phtml` to our own layout and make the folloing changes:

{% highlight php %}
<?php
/**
 * Magento
 *
 * NOTICE OF LICENSE
 *
 * This source file is subject to the Academic Free License (AFL 3.0)
 * that is bundled with this package in the file LICENSE_AFL.txt.
 * It is also available through the world-wide-web at this URL:
 * http://opensource.org/licenses/afl-3.0.php
 * If you did not receive a copy of the license and are unable to
 * obtain it through the world-wide-web, please send an email
 * to license@magento.com so we can send you a copy immediately.
 *
 * DISCLAIMER
 *
 * Do not edit or add to this file if you wish to upgrade Magento to newer
 * versions in the future. If you wish to customize Magento for your
 * needs please refer to http://www.magento.com for more information.
 *
 * @category    design
 * @package     base_default
 * @copyright   Copyright (c) 2006-2016 X.commerce, Inc. and affiliates (http://www.magento.com)
 * @license     http://opensource.org/licenses/afl-3.0.php  Academic Free License (AFL 3.0)
 */
?>
<?php
/**
 * Category view template
 *
 * @see Mage_Catalog_Block_Category_View
 */
?>
<?php
$_helper    = $this->helper('catalog/output');
$_category  = $this->getCurrentCategory();
$_imgHtml   = '';
if ($_imgUrl = $_category->getImageUrl()) {
    $_imgHtml = '<p class="category-image"><img src="'.$_imgUrl.'" alt="'.$this->escapeHtml($_category->getName()).'" title="'.$this->escapeHtml($_category->getName()).'" /></p>';
    $_imgHtml = $_helper->categoryAttribute($_category, $_imgHtml, 'image');
}
?>
<div class="page-title category-title">
    <?php if($this->IsRssCatalogEnable() && $this->IsTopCategory()): ?>
        <a href="<?php echo $this->getRssLink() ?>" class="link-rss"><?php echo $this->__('Subscribe to RSS Feed') ?></a>
    <?php endif; ?>
    <h1><?php echo $_helper->categoryAttribute($_category, $_category->getName(), 'name') ?></h1>
</div>

<?php echo $this->getMessagesBlock()->toHtml() ?>

<?php if($_imgUrl): ?>
    <?php echo $_imgHtml ?>
<?php endif; ?>

<?php if($_description=$this->getCurrentCategory()->getDescription()): ?>
    <div class="category-description std">
        <?php echo $_helper->categoryAttribute($_category, $_description, 'description') ?>
    </div>
<?php endif; ?>

<?php if($this->isContentMode()): ?>
    <?php echo $this->getCmsBlockHtml() ?>

<?php elseif($this->isMixedMode()): ?>
    <?php echo $this->getCmsBlockHtml() ?>
    <?php echo $this->getProductListHtml() ?>

<?php // That is the new part ?>
<?php elseif($this->isSubcategoryMode()): ?>
    <?php echo $this->getSubcategoryHtml() ?>

<?php else: ?>
    <?php echo $this->getProductListHtml() ?>
<?php endif; ?>

{% endhighlight %}

As you can see we inserted our call to `getSubcategoryHtml()` which now outputs our template meaning our subcategories.

### Download
You can find the complete module **[here](https://github.com/codedge/codedge-magento-subcategories)** on Github.
  

Hope you had fun reading this. If you spot any errors or have questions, drop me a message or write a comment.
