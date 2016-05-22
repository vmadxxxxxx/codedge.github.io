---
layout: post
title: Configuring Trusted Shops Review Toolkit Magento Extension
categories: [Magento-1.x, PHP, Web Development, Trusted Shops]
comments: true
tags: ['Magento', 'Trusted Shops', 'badge', 'review toolkit', 'configuration']
description: Configure Trusted Shops Review Toolkit for Magento.
feature: assets/media/2016-05-22/trusted_shops.png
excerpt: Trusted Shops released its brand new Review Toolkit for Magento. Additionally to the Trusted Shops Badge you can show product reviews.
---

Trusted Shops (TS) released its brand new Magento Extension [_Review Toolkit_](http://www.trustedshops.de/shop-info/neue-magento-extension-von-trusted-shops/) which is available on [Magento Connect](https://www.magentocommerce.com/magento-connect/catalog/product/view/id/31848/s/trusted-shopsr-reviews-toolkit/). The new extension makes it possible to display not only a valid TS Badge but also product reviews in your shop. These reviews will be fetched from Trusted Shop itself and can additionally be display in a Google Search.

The _Review Toolkit_ extension replaces the old [_Symmetrics TrustedRating_](https://www.magentocommerce.com/magento-connect/trusted-shops-with-trustbadger.html) module as TS confirmed me on [Twitter](https://twitter.com/trustedshops/status/733682411360391168).

> 1/2 Das Symmetrics_TrustedRating in seiner neuesten Fassung wird durch das neue Reviews Toolkit ersetzt.

I am going to talk about the custom configuration of the extension. 

## Configuration

Besides the default values already set by TS the is an _expert_ mode where profound shop manager can set more detailed settings - mostly for displaying and styling purposes.

All settings can be found in your Magento Admin in **System > Configuration > Services section > Trusted Shops**.

### Positioning the Trusted Shops Badge

When using an individual positioning instead of the preconfigured you need to create your own `div` container in which the badge is rendered into.

1. So first we put a container, i. e. `<div id="custom-ts-container"></div>`, in one of our templates. I used an already existing custom extension to create a template that is put into the footer of the page. 
2. Remember the given id `custom-ts-container` as we need to put this id into the Javascript snippet in Magento backend. I am going to show the full snippet in a while.
3. Apply CSS to `custom-ts-container`: I decided to go for a sticky position on the right border of the page which let the badge slide out when mouseover.

_New Javascript snippet_ for Magento backend:

{% highlight javascript %}
<!-- Trusted Shops Trustbadge Start -->
<script type="text/javascript">
    (function () {
        var _tsid = '%tsid%';
        _tsConfig = {
        'yOffset': '', // this is used for non-custom integrations, leave it blank
        'variant': 'custom_reviews', // need to be set to 'custom' or 'custom_reviews'
        'customElementId': 'custom-ts-container', // our div container id
        'trustcardDirection': 'topLeft', // put that to 'topLeft' as we will the badge on the right border
        'customBadgeWidth': '',
        'customBadgeHeight': '',
        'disableResponsive': 'false',
        'disableTrustbadge': 'false',
        'trustCardTrigger': 'click',
        'customCheckoutElementId': ''
        };
        var _ts = document.createElement('script');
        _ts.type = 'text/javascript';
        _ts.charset = 'utf-8';
        _ts.async = true;
        _ts.src = '//widgets.trustedshops.com/js/' + _tsid + '.js';
        var __ts = document.getElementsByTagName('script')[0];
        __ts.parentNode.insertBefore(_ts, __ts);
    })();
</script>
<!-- Trusted Shops Trustbadge End -->
{% endhighlight %}

This is the CSS to be applied to the div container:

{% highlight css %}
#custom-ts-container {
    position: fixed;
    right: 0;
    top: 50%;
    width: 6.5em;
    margin-top: -2.5em;
    -webkit-transition: width 1.5s;
    transition: width 1.5s;
    z-index: 2147483646 !important;
}

#custom-ts-container:hover {
    width: 22em;
}
{% endhighlight %}

Now we've got nicely positioned TS badge positioned vertically centered on the page. When mouseover it moves left to also 
show the rating of the page.

### Getting the Trusted Shops product reviews

Ideally you install the extension and - PENG! - it works, TS reviews are displayed on the product detail page. Unfortunately it did not work for me for a Magento installation prior to version 1.9. This is mostly because of some design changes in the layout files so I needed to do a little hack to layout file the TS extension is shipped with.

This might be due to some layout changes in the Magento shop I am working with - so please consider this an _optional_ step if you got problems with displaying the reviews.

The `layout.xml` for the reviews toolkit extension can be found in `app/design/frontend/base/default/layout/trustedshops.xml`. As you can see the reviews are being injected into a layout block called `product.info`:

{% highlight xml %}
<reference name="product.info">
    <block type="trustedshops/review_tab" name="trustedshops.review.tab" as="trustedshops_review_tab">
        <action method="addToParentGroup">
            <group>detailed_info</group>
        </action>
        <action method="setTitle" translate="title" module="trustedshops">
            <title>Trusted Shops Reviews</title>
        </action>
    </block>
</reference>
{% endhighlight %}

In my layout the appropriate block is called `product.info.tabs` - so I added just the same block under the one above:

{% highlight xml %}
 <!-- CUSTOM layout hack start -->
<reference name="product.info.tabs">
    <action method="addTab" translate="title" module="catalog">
        <alias>tsreviews</alias>
        <title>Trusted Shops Reviews</title>
        <block>catalog/product_view_attributes</block>
        <template>trustedshops/review_tab.phtml</template>
    </action>
</reference>
<!-- CUSTOM layout hack end -->
{% endhighlight %}

**Important:** You should not add this to the extensions layout file but rather use one of your own extensions. I just added it for demo purposes.

I hope this little guide helps you to set up the extension - pretty straight forward I think. Maybe you are also satisfied with the standard values.