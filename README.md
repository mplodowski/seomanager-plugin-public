# Renatio SEO Manager Plugin

Plugin adds SEO functionality to [OctoberCMS](http://octobercms.com). It supports CMS Pages, RainLab.Pages, RainLab.Blog out of the box. With one line of code can be attached to any OctoberCMS model. One robust solution for all your SEO needs.

## Features
* Support for CMS Pages, RainLab.Pages, RainLab.Blog
* Import default values from CMS Pages, RainLab.Pages, RainLab.Blog
* Open Graph support
* Robots meta configuration
* Canonical URLs
* 301 Redirects
* Easy edit robots.txt and .htaccess in CMS Settings
* Easy integration with other plugins

## Future plans
* Favicon Generator
* Google snippet preview
* Focus keywords check
* Page analysis

## Support

Please use [GitHub Issues Page](https://github.com/mplodowski/seomanager-plugin-public/issues) to report any issues with plugin.

> Reviews should not be used for getting support, if you need support please use the Plugin support link.

# Documentation
## Usage

After installation all CMS pages, Static pages, Posts, Categories will now display additional tabs with SEO and Open Graph fields.

To display meta information on frontend page you must only place single SEO component in theme layout head section.

```
[seoTags]
==
<!DOCTYPE html>
<html>
    <head>
        {% component 'seoTags' %}
    </head>
```

## Settings

Plugin ships with a settings page. Go to Settings and you will see a menu item SEO configuration listed under SEO bookmark.

Settings allow you to specify SEO Title prefix/suffix.

You can write common meta tags used in all pages e.g.

```
<meta name="author" content="Renatio">
<meta name="viewport" content="width=device-width, initial-scale=1">
```

You can enable/disable Open Graph tags output, specify Open Graph site name and Facebook Application ID.

You can edit robots.txt file and .htaccess file.

> Editing .htaccess file may break your site if not set up properly, so proceed with caution. This can be restricted by setting user permission.

## SEO fields

**Available SEO fields:**

#### SEO Title

Defines the title of a document. Title tags are often used on search engine results pages (SERPs) to display preview snippets for a given page, and are important both for SEO and social sharing. [Read more](https://moz.com/learn/seo/title-tag).

#### SEO Description

Meta descriptions are HTML attributes that provide concise explanations of the contents of web pages. Meta descriptions are commonly used on search engine result pages (SERPs) to display preview snippets for a given page. [Read more](https://moz.com/learn/seo/meta-description).

#### Meta Keywords

A series of keywords you deem relevant to the page in question.

#### Meta Robots

The robots meta tag is not the same as the file called robots.txt. You should use these two together. Both are used by the search engines like Yahoo and Google. [Read more](https://yoast.com/robots-meta-tags/).

#### Canonical URL

Canonicalization for SEOs refers to normalizing (redirecting to a single dominant version) multiple URLs. [Read more](https://moz.com/learn/seo/canonicalization).

#### 301 Redirect

Redirection is the process of forwarding one URL to a different URL. There are three main kinds of redirects: 301, 302, and meta refresh. [Read more](https://moz.com/learn/seo/redirection).

> More fields can be added on request.

## Open Graph fields

**Available Open Graph fields:**

#### OG Title

The title of your object as it should appear within the graph, e.g., "The Rock".

#### OG Description

A one to two sentence description of your object.

#### OG Type

The type of your object, e.g., "article".

#### Og Image

An image URL which should represent your object within the graph.

Read more about [Open Graph Protocol](http://ogp.me/).

> More fields can be added on request.

## Integration with models

SEO fields can be attached to any OctoberCMS model with single line of code. You just need to implement SEO Behavior in your model class like so:

```
public $implement = ['@Renatio.SeoManager.Behaviors.SeoModel'];
```

After implementing this behavior to your model class, SEO Manager will extend it with SEO fields and create relation to model.

To allow SEO Manager Plugin to recognize page with specific model attached you must passed it to the page view. This is most often done in component `onRun()` method like so:

```
$this->page['album'] = Album::find($id); // pass album record to page view
```

## Extending SEO fields

Plugin will fire `seo.extendSeoFields` event to allow for extensibility. This can be used to modify or add more SEO fields. You can listen for this event like so:

```
Event::listen('seo.extendSeoFields', function ($fields) {

    // modify or add more fields

    return $fields; // remember to return modified fields array
});
```

Similar approach can be used to extend Open Graph fields with `seo.extendOgFields` event.

Model fields are saved to database, so you must add columns to `renatio_seomanager_seo_tags` table before you can use them.

## Access SEO Tag before rendered on page

Plugin will fire `seo.beforeComponentRender` event to allow for extensibility. This can be used to access page with associated SEO Tag.

```
/*
 * Assign new seoTag for product page
 */
Event::listen('seo.beforeComponentRender', function ($component, $page) {
    if ($page->url == '/products/:slug') {
        $component->seoTag = $page->controller->vars['product']->seo_tag;
    }
});
```

## Console commands

Plugin will create three new artisan commands for working with console.

**seo:import-cms** command will import SEO from CMS pages.

**seo:import-static** command will import SEO from RainLab Static pages.

**seo:import-blog** command will import SEO from RainLab Blog posts and categories.