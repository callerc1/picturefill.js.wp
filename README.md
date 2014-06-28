Picturefill.wp 2
================

Picturefill.WP 2 is a simple and option-less plugin to serve HDPI and responsive images on a WordPress website.

This plugin parses post and page content replacing images with the syntax for the HTML5 `picture` element, then uses [picturefill.js](https://github.com/scottjehl/picturefill) to pollyfill the functionality on unsuported browsers.

Download
--------

The latest stable version of Picturefill.WP is available in the Wordpress.org plugin directory [http://wordpress.org/extend/plugins/picturefillwp/](http://wordpress.org/extend/plugins/picturefillwp/).

Details
-------

Picturefill.wp looks through `the_content` to find `<img>` elements like this:

```html
<img class="alignnone size-large wp-image-123" alt="Accessible alternate text for the image" title="A title that displays on hover" src="http://sitename.com/wp-content/uploads/2013/4/image-770x577.jpg" width="770" height="577" />
```

then replaces them with something like this:

```html
<picture>
<source srcset="http://sitename.com/wp-content/uploads/2014/05/image-700x525.jpg, http://sitename.com/wp-content/uploads/2014/05/image-1400x1050.jpg 2x" media="(min-width: 720px)">
<source srcset="http://sitename.com/wp-content/uploads/2014/05/image-300x225.jpg, http://sitename.com/wp-content/uploads/2014/05/image-600x450.jpg 2x" media="(min-width: 320px)">
<img alt="Image alt" class="alignnone size-large wp-image-1715" srcset="http://sitename.com/wp-content/uploads/2014/05/image-150x150.jpg, http://sitename.com/wp-content/uploads/2014/05/image-300x300.jpg 2x" />
</picture>
```

or something like this:

```html
<img alt="Image alt" class="alignnone size-large wp-image-1715" sizes="100vw" srcset="http://sitename.com/wp-content/uploads/2014/05/image-150x150.jpg 150w, http://sitename.com/wp-content/uploads/2014/05/image-300x225.jpg 300w, http://sitename.com/wp-content/uploads/2014/05/image-700x525.jpg 700w" />
```

###Heights and Widths and Breakpoints

One of the goals of this plugin is to be completely "plug and play" i.e. no setup and no options. Just turn it on and it works. To do this, the plugin relies on several Wordpress defaults and conventions.

####Wordpress Image Sizes

By default, Wordpress creates as many as 3 images of different sizes for each uploaded image ("large", "medium", and "thumbnail"), in addition to the "full" image size.

This plugin adds responsive breakpoints based on the width of the image. The largest available image will display unless the browser width is less than the image width + 20px, in which case the next size down is displayed.

To use this plugin most effectively, set the default image sizes ("large", "medium", and "thumbnail") to reflect useful breakpoints in your theme design.

####Wordpress Image Classes

The responsiveness of an image can be limited by adding the class `min-size-{image size}`. For example, an image with the class `min-size-medium` will not load an image smaller than size `medium`.

###Caching

To improve performance, especially in image heavy posts, the output of Picturefill.WP is cached with WordPress transients after it is generated. The cache will be refreshed automatically every time a post is updated or Picturefill.WP is updated. The cache can be manually refreshed by deactivating and reactivating Picturefill.WP from the plugins menu.

If you suspect that Picturefill.WP's caching is causing trouble with another plugin or theme feature, first try deactivating and reactivating Picturefill.WP. If problems persist, try lowering the priority for Picturefill.WP to be executed by adding the following to your functions.php file:

```php
remove_filter('the_content', array(Picturefill_WP::get_instance(), 'apply_picturefill_wp_to_the_content'), 11);
add_filter('the_content', array(Picturefill_WP::get_instance(), 'apply_picturefill_wp_to_the_content'), 9999);
```

If you still encounter problems with other plugins or theme features, you may want to disable caching all together. See the subsection on how to disable caching under the "Extending Picturefill.WP" section.

###Errors and Warnings

As of version 1.3.3 Picturefill.WP suppresses errors and warnings in parsing the DOM. Errors and warnings can now be collected via the `picturefill_wp_syntax_present_libxml_errors` and `picturefill_wp_get_images_libxml_errors` filters.

```php
add_filter('picturefill_wp_get_images_libxml_errors', 'handle_errors');

function handle_errors($errors){
  foreach($errors as $error){
    // Handle errors here.
  }
}
```

####Slow Loading on Activation

The first time a page or post is loaded after activating Picturefill.WP, new `@2x` size images will need to be created for the images in the post or page content. This can take several seconds and will take longer on image heavy posts. Once these images are created, posts should load at least as fast or faster than they do without the plugin.

If you are installing Picturefill.WP on a large and image heavy site, you may want to consider using another plugin like [Regenerate Thumbnails](http://wordpress.org/plugins/regenerate-thumbnails/) to create the new image sizes for existing posts and pages.

####500 or 504 server errors

These errors are related to the slow loading listed above. If the server reaches its timeout limit before it is finished processing new images, it will return a 500 or 504 error. Refreshing the page usually gives the server the time it needs to finish processing the images. On some image-heavy posts, it may take more than one refresh.

Extending and Customizing Picturefill.WP
------------------------

###Helper Functions

Picturefill.WP, as of version 1.3.0, includes a number of helper functions to simplify common customizations.

For extra safety, it's a good idea to wrap your code that targets Picturefill.WP in a conditional statement so it will only run if the plugin is active.

```php
if(defined('PICTUREFILL_WP_VERSION')){
  // Add Picturefill.WP specific code here.
}
```

####apply_picturefill_wp($filter [, $cache = ture, $priority = 11])

Applies Picturefill.WP to additional content blocks with via filters.

#####Example

Applies Picturefill.WP to the output text of the WordPress Text widget.

```php
apply_picturefill_wp('widget_text');
```

####disable_picturefill_wp_cache([$priority = 11])

Disables the transient cache.

####set_picturefill_wp_cache_duration($cache_duration_in_seconds)

Set the duration for the transient cache.

#####Example

Set the transient cache duration to one year (365 days). The default is 30 days.

```php
set_picturefill_wp_cache_duration(31536000);
```

####picturefill_wp_retina_only()

Removes browser-width resposiveness.

####picturefill_wp_remove_image_from_responsive_list($image_size)

Remove an image size from the list of those served.

#####Example

Only respond down to image size "medium" by removing the "thumbnail" size from the list of images served.

```php
picturefill_wp_remove_image_from_responsive_list('thumbnail');
```

####picturefill_wp_add_image_size($name [, $width = 0, $height = 0, $crop = false, $insert_before 'thumbnail'])

Create a new image size and add it to the list of responsive images.

#####Example

Add a new responsive image size in-between the "medium" and "large" image sizes.

```php
picturefill_wp_add_image_size('new_size', 550, 999, false, 'large');
```

####picturefill_wp_set_responsive_image_sizes($image_size_array)

Set the list of responsive images with an array.

#####Example

```php
$image_size_array = array('custom_small_size', 'thumbnail', 'extra-large');
/* All image sizes included in the $image_size_array
   must allready exist, either by default (thumbnail,
   medium, and large) or by the add_image_size function.
   Image sizes should be listed from smallest to largest
   and should not include '@2x' sizes, these will be
   added automatically. */
picturefill_wp_set_responsive_image_sizes($image_size_array);
```

####apply_picturefill_wp_to_post_thumbnail()

Apply Picturefill.WP to the `post_thumbnail_html` filter and use the `post_thumbnail` image size.

####minimize_picturefill_wp_output()

Reduce the html output of the `span` elements.


###Hooks

Like many WordPress themes and plugins, Picturefill.WP can be altered and extended with action and and filter hooks.

####Actions

* `picturefill_wp_updated`
* `picturefill_wp_before_replace_images`
* `picturefill_wp_after_replace_images`

####Filters

* `picturefill_wp_the_content_filter_priority`
* `picturefill_wp_content_html`
* `picturefill_wp_image_attributes`
* `picturefill_wp_image_attachment_data`
* `picturefill_wp_image_sizes`
* `picturefill_wp_source_list`
* `picturefill_wp_picture_attribute_string`
* `picturefill_wp_media_query_breakpoint`
* `picturefill_wp_media_query_resolution_query`
* `picturefill_wp_template_path`
* `picturefill_wp_{$template}_template_file_path`
* `picturefill_wp_{$template}_template_data`
* `picturefill_wp_{$template}_template`
* `picturefill_wp_the_content_output`
* `picturefill_wp_cache_duration`
* `picturefill_wp_syntax_present_libxml_errors`
* `picturefill_wp_get_images_libxml_errors`


Use With Other Plugins
----------------------

###Using Picturefill.WP with the [Advanced Custom Fields Plugin](http://wordpress.org/plugins/advanced-custom-fields/)

If you use [Advanced Custom Fields shortcodes](http://www.advancedcustomfields.com/resources/functions/shortcode/) in your post or page content, Picturefill.WP will work automatically. To use Advanced Custom Fields outside of `the_content` in theme files, apply Picturefill.WP to the `acf/format_value_for_api` filter.

```php
add_filter('acf/format_value_for_api', 'theme_function_picturefill_for_acf', 11, 3);

function theme_function_picturefill_for_acf($content, $post_id, $field){
  if(in_array($field['type'], array('textarea', 'wysiwyg', text))){
    return Picturefill_WP::get_instance()->cache_picturefill_output($content, $field['name']);
  }else{
    return $content;
  }
}
```

For image fields, you will need to wrap the image output in a custom filter.

In your theme file:
```php
<?php
$image_object = get_field('image');
$image_output = '<img src="' . $image_object['sizes']['medium'] . '" title="' . $image_object['title'] . '" alt="' . $image_object['alt'] . '" />';
echo apply_filters('theme_acf_image', $image_output, 'name_of_the_image_field');
?>
```

In functions.php:
```php
add_filter('theme_acf_image', 'theme_function_for_acf_image', 10, 2);

function theme_function_for_acf_image($content, $name_of_the_image_field){
  return Picturefill_WP::get_instance()->cache_picturefill_output($content, $name_of_the_image_field);
}
```

###Using Picturefill.WP with the [Infinite Scroll Plugin](http://wordpress.org/plugins/infinite-scroll/)

Picturefill.WP will replace any images that run through `the_content` filters with the picturefill HTML syntax, but picturefill.js is only run on page load and when the browser window is resized. It will not be run on any additional content added to the page after the page is loaded.

To work with an infinate scroll plugin, `window.picturefill();` will need to be added as a callback function, to be called anytime new content has loaded. In the case of the [infinite scroll plugin](http://wordpress.org/plugins/infinite-scroll/) linked at the top, there is a callback field in the plugin options where `window.picturefill();` can be added.

###Using Picturefill.WP with the [NextGen Gallery](http://wordpress.org/plugins/nextgen-gallery/)

NextGen Image Gallery uses custom database tables and custom folders for images, both of which cause trouble with Picturefill.WP. There are a few [ways to make things work with NextGen Gallery < 2.0](http://wordpress.org/support/topic/nextgen-2x-images), but no solution for version 2.0 or greater has come forward.

###Using Picturefill.WP with the [Woocommerce Shopping Cart](http://wordpress.org/plugins/woocommerce/)

The transient caching in Picturefill.WP causes a conflict with the way Woocommerce loads their shopping cart. [See the issue here](http://wordpress.org/support/topic/crashes-woocommerce-shopping-cart). Possible solutions include:

Disabling transient caching altogether.

In functions.php:
```php
if(defined('PICTUREFILL_WP_VERSION')){
  disable_picturefill_wp_cache();
}
```

Disabling transient caching on the cart shortcode.

In functions.php:
```php
if(defined('PICTUREFILL_WP_VERSION')){
  add_filter('the_content', 'do_not_cache_woocommerce_cart');
}

function do_not_cache_woocommerce_cart($content){
  if(has_shortcode($content, 'woocommerce_cart')){
    disable_picturefill_wp_cache();
  }
  return $content;
}
```

Telling Picturefill.WP to ignore the shopping cart.

In functions.php:
```php
if(defined('PICTUREFILL_WP_VERSION')){
  picturefill_wp_exclude_post_slug('cart');
}
```
