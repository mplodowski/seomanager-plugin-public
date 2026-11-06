# Renatio SEO Manager Plugin

Meta tags, Open Graph, JSON-LD, hreflang and XML sitemaps for [October CMS](https://octobercms.com) — ready for Google
and AI search.

**Demo URL:** https://october-demo.renatio.com/backend/backend/auth/signin  
**Login:** seomanager  
**Password:** seomanager

Supports CMS Pages, RainLab.Pages, RainLab.Blog and October CMS Tailor out of the box, and attaches to any October
CMS model with one line of code.

## Features

* SEO and Open Graph tabs for CMS pages, RainLab.Pages, RainLab.Blog, Tailor entries and any model
* Import of existing SEO values from CMS pages, static pages and blog posts
* Live Google snippet and Facebook / X share card previews in the backend forms
* Content analysis with a focus keyword, a checklist and a 0-100 score, per record and for the whole site
* Robots meta, canonical URLs and 301 redirects, including automatic redirects for renamed records
* A 404 log with the closest existing page for each missing address
* JSON-LD: Organization or LocalBusiness, WebSite, BreadcrumbList, Article/BlogPosting, and FAQPage, Product and Event
  mapped from Tailor fields
* Social profile links with brand icons, from the same data the structured data uses
* Hreflang tags and an XML sitemap with alternates for multisite
* llms.txt and llms-full.txt for language models and agents
* IndexNow submissions to Bing, Yandex, Seznam and Naver
* SEO Health page, dashboard widget and `seo:doctor` reporting misconfigurations that fail silently
* robots.txt editor with an AI crawler policy, and an optional .htaccess editor
* October's native multisite and translations; RainLab.Translate is still read as legacy
* Backend in English, Czech, Dutch, French, German, Italian, Polish, Brazilian Portuguese, Russian and Spanish

## Requirements

- October CMS 4.4 or newer
- PHP 8.2 or newer, with the `dom` and `xml` extensions
- `spatie/laravel-sitemap` 7.2+ or 8.0.1+ (8.x needs PHP 8.4) and `league/html-to-markdown` 5.1+, installed by Composer

## Why is this a paid plugin?

Something that is free has little or no perceived value. Users do not commit to free products and only use them until
something else that looks nice and is free comes along. When I invest my time in the development of a new plugin I commit to
supporting and maintaining it. I ask my customers to do the same. I do not make money from this plugin by
advertisements, upgrades or additional services like hosting or setup.

Did you know that 30% of your purchase or donation goes to help fund the October Project?

My plugins take many hours to develop (40-120+) and even more hours to document and maintain. My paid plugins have to
pay for both this time, and the time I am spending on free plugins and less successful paid plugins. This means that it
will take even a successful plugin years to become profitable. Please consider buying an extended license if you want me
to continue to maintain these plugins for the very small fee I ask in return or hire me for adding functionality that
you feel is missing but valuable.

## Like this plugin?

If you like this plugin, give this plugin a Like or Make donation with [PayPal](https://www.paypal.me/mplodowski).

## My other plugins

Please check my other [plugins](https://octobercms.com/author/Renatio).

## Support

Please use [GitHub Issues Page](https://github.com/mplodowski/seomanager-plugin-public/issues) to report any issues with
plugin.

> Reviews should not be used for getting support, if you need support please use the Plugin support link.

Icon made by [Darius Dan](https://www.flaticon.com/authors/darius-dan)
from [www.flaticon.com](https://www.flaticon.com/).

# Documentation

## Usage

CMS pages, static pages, blog posts and categories get SEO and Open Graph tabs after installation (on RainLab.Pages 3
they are tabs of the static page Settings popup). Place the `seoTags` component in the head of your layout:

```twig
[seoTags]
==
<!DOCTYPE html>
<html>
    <head>
        {% component 'seoTags' %}
    </head>
```

To prefill the SEO fields from what the pages already hold, run `seo:import-cms`, `seo:import-static` or
`seo:import-blog`.

### Where the tags come from

A page that renders a Tailor entry with the SEO mixins, or a model with the `SeoModel` behavior, takes its tags from
that record; any other page uses its own SEO tab.

- A blank field of the record falls back to the same field of the CMS page, except the canonical URL, the 301 redirect,
  `og:type` and the focus keyword. A blank **SEO Title** falls back to the record's `title` or `name`.
- The record is the one the address names: first the one a component looks up by a URL parameter (`blogPost` or
  `section` bound to `:slug`), then a page variable holding a parameter's value (`:slug` and its `slug`), then the
  record a component shows on its own. Only a page without URL parameters takes the first record with SEO fields among
  its variables, so a featured post in a sidebar never takes over the page.
- The hreflang links describe the same record, and the sitemap applies the same `noindex` fallback.

Behind a CDN, a reverse proxy or a full-page cache set `LINK_POLICY=force` in `.env`. With the default `detect` October
builds the canonical, `og:url` and hreflang addresses from the host of the request, which an `X-Forwarded-Host` header
can change.

## October CMS Tailor

Add the two mixins to the `fields` of a blueprint, then run `php artisan tailor:migrate`:

```yaml
_seo_meta_fields:
    type: mixin
    source: Renatio\SeoManager\MetaFields

_seo_og_fields:
    type: mixin
    source: Renatio\SeoManager\OgFields
```

Run `tailor:migrate` after every plugin update that changes the mixins: `october:migrate` does not touch Tailor
tables, and saving an entry fails with `Unknown column` until it runs. The **Tailor blueprint columns** check on SEO
Health names the tables that are behind and runs the migration from a button.

## Your own models

Add the behavior to the model:

```php
public $implement = ['@Renatio.SeoManager.Behaviors.SeoModel'];
```

The next `php artisan october:migrate` adds the SEO columns to its table (failures go to the log, not the console). To
add them right away, or when `add_seo_columns` is `false` in `config/renatio/seomanager/migrate.php`:

```bash
php artisan seo:migrate-tables --force
```

Without the columns nothing entered in the SEO tab is stored; the **SEO columns** check on SEO Health names such tables
and adds the columns from a button.

Publish the record to the page, with an attribute matching a URL parameter (`/albums/:slug` and the album's `slug`):

```php
public function onRun()
{
    $this->page['album'] = Album::where('slug', $this->param('slug'))->first();
}
```

Or assign it yourself before the component renders:

```php
Event::listen('seo.beforeComponentRender', function ($component, $page) {
    if ($page->url === '/products/:slug') {
        $component->seoTag = $page->controller->vars['product'];
    }
});
```

`$component->seoTag` is a copy of the record: changes made to it never reach the database. Titles and descriptions are
escaped, nothing else is, so assign only content you would put on the page yourself.

## Settings

**Settings > SEO > SEO Manager**:

| Tab | Holds |
|---|---|
| General | site name, **Language and region** (multisite only, e.g. `de-CH`), **Default language version (x-default)**, title prefix and suffix, **Common meta tags**, **Allowed redirect hosts**, **Redirect renamed records**, **404 log** and **Keep entries for**, **Live content analysis** |
| Open Graph | the switch, Facebook Application ID, Twitter/X site handle, default image |
| Organization | the JSON-LD switch, Schema.org identity (Organization or a LocalBusiness type), logo, contact data, address, founding date, coordinates and opening hours |
| Social profiles | one row per profile: **Address**, **Service** (picks the icon), optional **Name** |
| Sitemap | the switch, **Rebuild** frequency and hour, **Excluded paths**, the IndexNow section |
| llms.txt | the llms.txt and llms-full.txt switches, summary, links, **Add the pages of the sitemap** |
| robots.txt | the editor and the **AI crawler policy** |
| .htaccess | the switch and the editor, only where the server reads .htaccess |

Fields with a fixed format (locale, country, X handle, Facebook ID, telephone, prefix and suffix) are validated, and the
country code and handle are normalized (`pl` → `PL`, `acme` → `@acme`). robots.txt is read from and written to the
web root (`public` once it holds `index.php`, otherwise a file in the project root is kept); .htaccess keeps a project
root file while `public` has none. Neither is ever overwritten with empty content.

> **Common meta tags** are output unescaped on every page, so they can carry scripts. They need their own permission.

### Multisite

Each site has its own settings: pick it in the backend site switcher. A site never saved reads the settings of its site
group, and **Reset to default** returns it to them rather than to the plugin defaults. Its locale is the exception: a
new site reports its own October locale until a region is set.

Shared by the whole installation, and labelled as such: robots.txt with the AI crawler policy, .htaccess and its switch,
the allowed redirect hosts, x-default, the sitemap rebuild schedule, the 404 log retention and the IndexNow key.

### The .htaccess tab

The tab appears where `SERVER_SOFTWARE` names Apache or LiteSpeed, and editing stays off until switched on there.
With `cms.safe_mode` on the editor is read-only and the panel never writes the file.
Decide it yourself in `config/renatio/seomanager/htaccess.php`:

```php
return [
    'tab' => true, // or false; null detects the server
];
```

### Permissions

| Permission | Guards | Default roles |
|---|---|---|
| Manage SEO settings | the settings page, every field not listed below, **Generate a new key** | Developer, Publisher |
| View SEO Health and run its checks | SEO Health, its widget, sitemap rebuild, deleting rename redirects and 404 entries | Developer, Publisher |
| Change robots.txt and the AI crawler policy | the robots.txt editor, its reset and the AI crawler policy | Developer |
| Change the allowed redirect hosts | the external hosts a 301 may point to | Developer |
| Change common meta tags | the raw HTML in the head of every page | Developer |
| Change the .htaccess file | the .htaccess switch and editor | Developer |
| Add the missing SEO columns | the SEO Health buttons that alter other plugins' tables and run `tailor:migrate` | Developer |
| Delete the static sitemap file | the SEO Health button that deletes a file from the web root | Developer |

A field the user lacks the permission for is left out of the form. October locks its built-in roles, so give a
Developer-only permission to an editor through a custom role in **Settings > Administrators > Roles**.

## SEO fields

| Field | Notes |
|---|---|
| SEO Title | the document title; prefix and suffix from the settings are added |
| SEO Description | the snippet in search results |
| Focus Keyword | the phrase the content analysis checks for |
| Meta Robots | index / follow switches; **Meta Robots Advanced** takes other directives, e.g. `noarchive, max-snippet:150` (not `all` or `none`) |
| Canonical URL | a path from the root or an absolute http(s) URL |
| Meta Keywords | ignored by Google and Bing, kept last |
| 301 Redirect | a relative path or an absolute URL on a site host or an allowed redirect host |

On a CMS page, October's own **Meta** section fills whatever the SEO popup leaves blank. The 301 redirect needs the
`seoTags` component on the page or its layout; it is skipped when it points at the current URL, loops back through a
rename redirect, or on error pages. A non-GET request to a target on a site host is redirected with 308, so its body
survives.

The canonical tag names the record's own address (a Tailor entry or blog post at `/notes/01` is canonical `/notes/1`)
and drops the query string except the page number of a paginated listing: the parameters in `query_params` of
`config/renatio/seomanager/canonical.php` (default `page`), kept as positive integers, with page 1 named without it. The JSON-LD page
address follows it; breadcrumbs keep the path only. A canonical naming another address takes the page out of the
sitemap, llms.txt and IndexNow, while SEO Health still audits it; a record kept per site (a Tailor entry, a blog post
per site) also leaves the hreflang set of its translations.

### Error responses

- Any status of 400 or above gets `noindex, nofollow`, except 503 (maintenance mode).
- The theme's `404` and `error` pages and any 404 or 410 also drop the canonical (unless one is filled in), the
  alternates, the Open Graph and Twitter tags, and the `BreadcrumbList` and `Article` schemas.

## Open Graph

**OG Title**, **OG Description**, **OG Type** and **OG Image** fall back to the SEO title, the SEO description, an
automatic type (`article` for a record with a publish date, `website` otherwise) and the record's own image. Add a type:

```php
Event::listen('seo.extendOgTypes', function (array &$types) {
    $types[] = 'music.playlist';
});
```

**Locale.** `og:locale` comes from the active site and `og:locale:alternate` from the other sites of its group. A bare
language gets its usual territory (`de` → `de_DE`, `pt-br` → `pt_BR`). Two sites serving one language to two countries
set **Language and region** (`de-DE`, `de-CH`), which also drives `hreflang`. Change the map in
`config/renatio/seomanager/og_locales.php` (only the keys you change) or per language:

```php
Event::listen('seo.extendOgLocales', function (array &$locales) {
    $locales['de'] = 'de_AT';
});
```

**Image.** Without an **OG Image** the record's own image is used, then the default image from the settings. The
attributes or `attachOne` / `attachMany` relations `banner`, `featured_images`, `image`, `images`, `photo`, `photos`,
`cover` and `thumbnail` are tried in order, then every other attachment the model declares. Only public `jpg`, `jpeg`,
`png`, `gif`, `webp` and `avif` files count; width and height are known only for Media Library images. Set `sources` or
`'search_attachments' => false` in `config/renatio/seomanager/record_image.php`, or add a name per model:

```php
Event::listen('seo.extendImageSources', function (array &$sources, Model $model) {
    if ($model instanceof Album) {
        $sources[] = 'cover_photo';
    }
});
```

## Backend preview

The SEO tab opens with the Google result, the Open Graph tab with the Facebook and X cards, redrawn as you type. An
empty field shows in grey what the page would use instead. Title and description are measured in pixels (580 and 920,
a desktop result). The address follows the page that serves the record and the **Slug** field; the favicon comes from
the theme's `assets/images/favicon.*`, `assets/img/favicon.*`, `assets/favicon.*` or the web root.

The preview cannot show what a blank field inherits from the CMS page, and an unsaved record shows the default image.

## Content analysis

Every SEO tab carries a **Content analysis** panel scoring the form as it stands. It refreshes as you type; with **Live
content analysis** off it shows an **Analyze again** button instead. CMS pages get the **Focus Keyword** field but no
panel. Keyword checks are skipped without a keyword; the keyword is matched ignoring case and accents but not stemmed.

| Check | Passes when |
|---|---|
| Keyword in the title / meta description / a heading / the address / the opening paragraph | it is found there (prefix and suffix count; hyphens in the slug read as spaces) |
| Keyword density | 0.5-2.5 % of the words; skipped under 100 words |
| Title length | 30-60 characters |
| Meta description length | 50-160 characters |
| Content length | at least 300 words |
| Heading structure | at most one H1 and no skipped level (a missing H1 passes) |
| Image alt text | every image has alt text |

The score is the share of the applicable checks' weight: **Good** from 80, **Fair** from 50, **Poor** below. An empty
body fails **Content length** and warns on **Heading structure**; a record whose body field cannot be found is skipped
with a hint.

### Where the body comes from

1. the posted form;
2. the `seoContent` key of the Tailor blueprint:

   ```yaml
   handle: Blog\Article
   type: entry
   seoContent: [lead, body]
   ```

3. `sources` in `config/renatio/seomanager/content_text.php` (default `markup`, `content_html`, `content`, `body_html`,
   `body`, `text`, `description`, `intro`, `excerpt`);
4. the `seo.resolveContentText` event, fired on every record with what the steps above found:

   ```php
   Event::listen('seo.resolveContentText', function (string &$text, mixed $record, array $posted) {
       if ($record instanceof Album) {
           $text = $record->notes;
       }
   });
   ```

### Custom checks

```php
use Renatio\SeoManager\Classes\Analysis\Check;
use Renatio\SeoManager\Classes\Analysis\Content;
use Renatio\SeoManager\Classes\Analysis\Result;

class PlaceholderText extends Check
{
    public function key(): string
    {
        return 'acme_placeholder';
    }

    public function run(Content $content): Result
    {
        if ($content->document()->isEmpty()) {
            return $this->skip('no_content');
        }

        return str_contains(mb_strtolower($content->document()->text()), 'lorem ipsum')
            ? $this->problem('found_text')
            : $this->ok();
    }
}

Event::listen('seo.extendAnalysisChecks', function (array &$checks) {
    $checks[] = new PlaceholderText; // an instance, not a class name
});
```

A check returns `ok()`, `warning()`, `problem()` or `skip()` and may override `weight()` (1). A check that throws is
dropped. Labels come from `app/lang/renatio/seomanager/<locale>/lang.php`, which October merges over the plugin's:

```php
return [
    'analysis' => [
        'acme_placeholder' => 'No placeholder text',
        'acme_placeholder_ok' => 'The content holds no lorem ipsum.',
        'acme_placeholder_found_text' => 'The content still holds lorem ipsum.',
    ],
];
```

## JSON-LD structured data

`seoTags` outputs `<script type="application/ld+json">` blocks; switch them on the **Organization** settings tab, which
also warns about missing data. The **Structured data** tab of SEO Health previews the output for any address and links
to the Schema.org validator and the Rich Results Test.

| Schema | Output |
|---|---|
| Organization | when the site name is set; a LocalBusiness type adds coordinates and opening hours; empty fields are left out |
| WebSite | when the site name is set |
| BreadcrumbList | every page but the home page, from the URL path; intermediate levels only where a CMS page serves them |
| Article / BlogPosting | records with `title` and `published_at`; Tailor entries and blog posts are `BlogPosting` |
| FAQPage, Product, Event | mapped from Tailor fields, see below |

| Article property | Resolution order |
|---|---|
| headline | `meta_title` → `title` (max 110 chars) |
| description | `meta_description` → `excerpt` → `content` (first 200 chars), as plain text |
| image | `og_image` → the record image (see Open Graph) |
| author | `user` relation (full name) → `author` string → `author` relation |
| publisher | the Organization, with name, URL and logo |
| mainEntityOfPage | Canonical URL → page address |
| datePublished / dateModified | `published_at` / `updated_at` → `published_at` |

### Schemas from Tailor fields

Map entry fields under `seoSchema` in the blueprint. A value names a field, `field.attribute` reaches through a
relation, `=` starts a constant; `items` names a repeater:

```yaml
handle: Shop\Product
type: entry
seoSchema:
    -
        type: Product
        name: title
        description: intro
        image: photo
        sku: sku
        brand: brand_name
        offers:
            price: price
            priceCurrency: =PLN
            availability: stock_status
        aggregateRating:
            ratingValue: rating
            reviewCount: reviews_count
    -
        type: FAQPage
        items: faq_items
        question: question
        answer: answer
```

| Type | Written only when it has |
|---|---|
| `FAQPage` | at least one item with a question and an answer |
| `Product` | `name` and `offers` or `aggregateRating`; an offer needs a numeric `price` and a `priceCurrency` |
| `Event` | `name`, `startDate` and a `location` (an address or a stream URL) |

Dates are written as ISO 8601, availability only as Schema.org values (`InStock`, `OutOfStock`, ...), a decimal comma
is converted. Google shows FAQ rich results only for government and health sites. The **Schemas from Tailor fields**
check on SEO Health reports unknown types, missing required properties and unknown fields.

Map records without a blueprint, or add any schema:

```php
Event::listen('seo.extendSchemaMapping', function (array &$mappings, mixed $record) {
    if ($record instanceof Post) {
        $mappings[] = ['type' => 'FAQPage', 'items' => 'faqs', 'question' => 'question', 'answer' => 'answer'];
    }
});

Event::listen('seo.extendJsonLd', function (array &$schemas, mixed $seoTag, Settings $settings, bool $missing) {
    $schemas[] = ['@context' => 'https://schema.org', '@type' => 'Event', /* ... */];
});
```

## Social profiles

The **Social profiles** tab feeds `sameAs` of the Organization schema and the `socialProfiles` component, each site its
own:

```twig
[socialProfiles]
only = "facebook,instagram"
==
{% component 'socialProfiles' %}
```

| Property | Default | Description |
|---|---|---|
| `only` | | only these services, comma separated |
| `except` | | leave these services out |
| `target` | `_blank` | link target; empty opens in the same tab |
| `class` | `social-profiles` | class of the list and stem of the element classes |
| `iconSize` | `24` | icon width and height in pixels |
| `iconClass` | | an icon font instead of the SVG; `fab fa-:icon` renders `fab fa-facebook` |

Links carry an inline SVG brand icon and an `aria-label`. To change the markup, copy the partial to
`themes/<theme>/partials/socialProfiles/default.htm`; each profile has `url`, `icon`, `name`, `path` and `viewBox`:

```twig
{% for profile in __SELF__.profiles %}
    <a href="{{ profile.url }}" aria-label="{{ profile.name }}">
        <svg viewBox="{{ profile.viewBox }}" width="20" height="20" aria-hidden="true">
            <path d="{{ profile.path }}"/>
        </svg>
    </a>
{% endfor %}
```

Add icons, or profiles kept elsewhere (`url` and `icon` are enough):

```php
Event::listen('seo.extendSocialIcons', function (array &$icons) {
    $icons['goldenline'] = ['name' => 'GoldenLine', 'viewBox' => '0 0 24 24', 'path' => 'M12 0C5.4 0 ...'];
});

Event::listen('seo.extendSocialProfiles', function (array &$profiles, Settings $settings) {
    $profiles[] = ['url' => 'https://www.youtube.com/@acme', 'icon' => 'youtube'];
});
```

Icons: [Simple Icons](https://simpleicons.org) (CC0); LinkedIn and the globe from
[Bootstrap Icons](https://icons.getbootstrap.com) (MIT).

## Hreflang tags

With [multisite](https://docs.octobercms.com/4.x/cms/multisite/multisite.html) and two or more sites in a set,
`seoTags` outputs `<link rel="alternate" hreflang="…">` for each site plus `x-default`, which points to the primary site
unless **Default language version (x-default)** on the General tab names another. The links carry no query string
beyond the page number, like `canonical` and `og:url`.

A record is linked under each site's own version of it: Tailor entries, blog posts kept per site, and records of any
model with October's `Multisite` trait sharing a `site_root_id`. A site without a version, or one set to `noindex`, gets
no link. The sitemap alternates use the same sets.

Which sites form a set:

- **Sites running one theme join across groups**, unless a language is claimed in two of their groups (two brands on
  one theme, or `de` on a German and a Swiss domain); then each group keeps its own set. Set a region (`de-DE`,
  `de-CH`) to join them.
- **Groups are all or nothing.** Without any groups every enabled site is linked; once one site has a group, a site
  alone in its group or without one is linked only to sites running its theme.
- **Every site needs its own language.** Sites claiming the same language drop out of the set; SEO Health names the
  language on its Sitemap tab.

Narrow the sites (keep the result reciprocal), or edit the links of a page (a set left with fewer than two links is
dropped):

```php
Event::listen('seo.alternateSites', function (SiteCollection &$sites) {
    $sites = $sites->inSiteGroup();
});

Event::listen('seo.extendHreflang', function (array &$links, Controller $controller) {
    $links[] = ['locale' => 'fr', 'url' => 'https://example.com/fr/page'];
});
```

## XML sitemap

Enable it on the **Sitemap** settings tab to serve `/sitemap.xml` per site (under its route prefix), with `xhtml:link`
alternates and a stylesheet at `/sitemap.xsl`. It lists CMS pages, static pages, blog posts and categories, Tailor
entries on a page with a `section` component (the **Default View** page wins), and records of any plugin on a page with
URL parameters. `lastmod` comes from `updated_at`, and addresses of a site without its own URL are built on `APP_URL`.

Left out: hidden pages, `noindex`, records with a 301 redirect, `/404`, `/error`, and the **Excluded paths**: one
pattern per line, without a host or query string, `*` for any part, matched without the route prefix:

```
/api/*
/search
/feed
```

When every layout of the theme carries `seoTags` (a subclass registered under another code counts), a CMS page with no
layout and no `seoTags` of its own (a feed, an export, a JSON endpoint) is left out of the sitemap, llms.txt, IndexNow
and the SEO Health audit, along with the Tailor entries and blog records it renders. A theme with a layout lacking the
component, or with the component in a partial, keeps every page.

A map over 50 000 URLs or 50 MB is split into `sitemap-1.xml`, `sitemap-2.xml`, … behind an index at `/sitemap.xml`.
With the sitemap off, `/sitemap.xml` is left to the CMS, so a theme page can serve it.

### Rebuilding

The map is cached in `storage/app/seomanager` and rebuilt:

- by the scheduler at the **Rebuild** frequency (every hour, every day or Sunday at the chosen hour in
  `app.schedule_timezone`, or never; default daily at 03:00) — this needs October's cron entry:

  ```bash
  * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1
  ```

- by the **Rebuild sitemap** button on SEO Health, through the queue (inline on `sync` and `null`);
- by the first request for a map older than 48 hours (two weeks with a weekly rebuild).

Other requests wait up to ten seconds and get the previous map, or a `503` with `Retry-After` on a cold cache. A site
whose map fails keeps its previous one and is logged; `seo:sitemap` lists it and exits with a failure. Give the queue
connection a `retry_after` longer than a full rebuild. Maps and llms documents answer `If-Modified-Since` with `304`.

```bash
php artisan seo:sitemap          # rebuild every enabled site
php artisan seo:sitemap --clear  # clear the cache
```

### Pages with URL parameters

A page such as `/products/:slug` is filled from October's page finder (the `cms.pageLookup` events behind a menu
item's **Page** field), so any plugin implementing it is listed. Plugins answering only RainLab.Pages'
`pages.menuitem` events should listen to both:

```php
Event::listen(['cms.pageLookup.getTypeInfo', 'pages.menuitem.getTypeInfo'], ...);
```

Records read this way are listed even when set to `noindex`. A page no type can list is reported on SEO Health; set
it to `noindex` if it does not belong in the map.

### Extending the sitemap

```php
use Spatie\Sitemap\Sitemap;
use Spatie\Sitemap\Tags\Url;

Event::listen('seo.extendSitemap', function (Sitemap &$sitemap) {
    foreach (Product::where('is_published', true)->get() as $product) {
        $sitemap->add(Url::create(Cms::pageUrl('product/show', ['slug' => $product->slug]))
            ->setLastModificationDate($product->updated_at));
    }
});

// Read blog posts through your own Post subclass (e.g. one with the Multisite trait)
Event::listen('seo.blogPostModel', fn () => \Acme\Blog\Models\Post::class);
```

Add the address as a `DescribedUrl` to give llms.txt and llms-full.txt its title, description and body (Markdown,
read only when llms-full.txt is written). A plain `Url` is listed by its address alone.

```php
use Renatio\SeoManager\Classes\DescribedUrl;

$sitemap->add(DescribedUrl::create($url)->describe($product->name, $product->summary, fn () => $product->description));
```

## llms.txt and llms-full.txt

Enable them on the **llms.txt** settings tab. `/llms.txt` ([llmstxt.org](https://llmstxt.org)) is built from the site
name, the **Summary** and the **Links** (rows sharing a section are grouped; a URL starting with `/` gets the site's
prefix). **Add the pages of the sitemap** appends up to 500 sitemap addresses under `## Pages`, newest first.

```markdown
# Acme Studio

> Acme Studio designs and builds furniture for public spaces.

## Docs

- [Pricing](https://acme.test/pricing): What it costs
```

`/llms-full.txt` writes out the text of the same 500 pages:

```markdown
## Steel heads

Source: https://acme.test/products/steel-heads

Forged, not cast.
```

A Tailor entry publishes only the fields marked `llms: true` in its blueprint (or its `groups:`), of type `markdown`,
`textarea`, `text` or `richeditor`; other pages without readable body (CMS pages, static pages, page finder records)
get their heading, address and description, in the language of each site. Blog posts are published in full.

```yaml
fields:
    content:
        type: richeditor
        llms: true
    internal_notes:
        type: markdown
```

Both are cached and rebuilt like the sitemap, on its **Rebuild** frequency, with the **Rebuild llms.txt** button or
`seo:llms`. An expired document is served while a worker rebuilds it; saving the settings expires the site's documents.
Remove any static `llms.txt` or `llms-full.txt` from the web root (the **Static llms.txt file** check finds them).

```php
Event::listen('seo.extendLlmsTxt', function (array &$sections) {
    $sections[] = ['name' => 'Products', 'links' => [
        ['title' => 'Hammer', 'url' => 'https://acme.test/hammer', 'description' => 'Forged steel'],
    ]];
});

Event::listen('seo.extendLlmsContent', function (string|false &$content, array $descriptor) {
    if (str_starts_with($descriptor['url'], 'https://acme.test/internal/')) {
        $content = false; // leaves the page out of llms-full.txt
    }
});
```

`$descriptor` holds `url`, `title`, `description` and `lastmod`.

## IndexNow

Tells Bing, Yandex, Seznam and Naver (not Google) that an address changed, through `api.indexnow.org`. Turn it on per
site in the IndexNow section of the **Sitemap** tab. Saving generates one key shared by all sites, served at
`/{key}.txt`; **Generate a new key** replaces it everywhere and invalidates the old one.

```bash
php artisan seo:indexnow https://example.com/page   # one address
php artisan seo:indexnow --all                      # every sitemap address of every site with IndexNow on
```

**Submit on save** announces Tailor entries, blog posts and categories, CMS pages and static pages saved outside the
console, by the sitemap rules, including the old address after a slug change and records just unpublished, set to
`noindex` or deleted. Records published with a future date are announced when it passes (`seo:indexnow --scheduled`,
every 15 minutes, needs the cron entry). A save made outside the backend, like a view counter, is announced only when
it creates the record or changes one of `frontend_fields`, an SEO field or a Tailor content field; relations and
RainLab.Translate fields saved there go unannounced, so extend the list or call IndexNow yourself. Submissions are
queued (3 attempts, retried after 60 and 300 seconds), or run after the response on `sync`, `null`, `deferred` and
`background`.

`seo:indexnow` takes only an `http`/`https` address on the host of a site with IndexNow on; `--all` and `--scheduled`
take none and cannot be combined. It exits `1` on bad input, on a rejected or unreachable submission (the HTTP status
or cURL error is printed) and on a site without a key yet; an address still inside the debounce period exits `0`. On a
real queue driver the addresses are queued and the outcome is only logged.

Override any of these in `config/renatio/seomanager/indexnow.php` (only the keys you change):

```php
return [
    'endpoint' => 'https://api.indexnow.org/indexnow',
    'debounce' => 3600,       // seconds an accepted address is not submitted again
    'bulk_threshold' => 100,  // a request saving more records is skipped as an import
    'timeout' => 10,
    'frontend_fields' => ['title', 'slug', 'content', 'published', /* … */], // columns a frontend save is announced for
];
```

## Redirects after a rename

With **Redirect renamed records** (General tab, on by default) a changed `slug` of a Tailor entry or a RainLab.Blog post
or category leaves a 301 from the old address, built from the page serving it on each site. In a structure the whole
branch is redirected. Custom `SeoModel` models, changes of category, parent or date, and slugs translated by
RainLab.Translate get no redirect.

Chains are shortened (`a → c`, `b → c`), an address another record moves onto stops redirecting, and deleting the
record drops its redirects (trashing keeps them). Addresses over 255 characters are logged instead. The redirect is
served on a 404 response, so it also covers routes like `/blog/:slug`. Review and delete them on **SEO Health →
Redirects**.

With [Vdlp.Redirect](https://octobercms.com/plugin/vdlp-redirect) installed, the plugin also fires
`vdlp.redirect.toUrlChanged` so its rules to the old address follow the rename.

## The 404 log

**SEO Health → 404 log** lists the addresses the site answered 404 at, most requested first, with hits, last hit, the
first referrer and the closest page from the sitemap. Switch it per site with **404 log** on the General tab and set
**Keep entries for** (7, 30 or 90 days, all sites).

- An address is listed from its second request within a day.
- Not logged: addresses a redirect answers, non-GET requests, paths with a file extension other than `.html`/`.htm`,
  and paths under `/combine`, `/modules`, `/plugins`, `/themes` or `/storage`. Query strings are dropped.
- At most 1000 listed and 1000 pending addresses per site; the scheduler prunes daily.

With Vdlp.Redirect installed, each row has a **Create redirect** button that opens its form prefilled.

## robots.txt and AI crawlers

The **robots.txt** tab edits the file and sets an **AI crawler policy**: *Leave alone*, *Allow AI search, block
training*, *Allow all*, *Block all* or *Custom per crawler*.

| Purpose | Crawlers |
|---|---|
| search (answers questions, sends traffic) | `OAI-SearchBot`, `ChatGPT-User`, `Claude-SearchBot`, `Claude-User`, `Bravebot`, `PerplexityBot`, `Amazonbot` |
| training | `GPTBot`, `ClaudeBot`, `Google-Extended`, `Applebot-Extended`, `CCBot`, `Bytespider` |

The policy is written into a managed block; your own rules stay untouched, and the tab warns when they already define a
group for one of these crawlers:

```
# BEGIN SEO Manager AI crawlers
User-agent: GPTBot
Disallow: /
# END SEO Manager AI crawlers
```

```php
Event::listen('seo.extendAiCrawlers', function (array &$crawlers) {
    $crawlers[] = ['agent' => 'AcmeBot', 'vendor' => 'Acme', 'purpose' => 'training'];
});
```

## Multisite and translations

| Content | Native multisite | RainLab.Translate |
|---|---|---|
| Tailor entries | per site (text fields `translatable: true`) | n/a |
| Models with `SeoModel` | per site with the `Multisite` trait | per locale when the model declares `$translatable` |
| CMS pages and static pages | per theme, per locale in the **Translate SEO** popup | per locale in the **Translate SEO** popup |

The behavior adds the SEO fields to `$translatable` only where the model declares it, so October's `Translatable`
trait keeps them per locale too. CMS page SEO lives in the page file: sites sharing a theme share it. Without
RainLab.Translate the **Translate SEO** popup has a tab per language of the sites serving the theme. A CMS page title
falls back from the translated SEO title to the translated meta title to the page title.

**RainLab.Translate is legacy**: still read, no longer developed. The **RainLab.Translate data** check on SEO Health
lists what only it still holds and the command that moves it:

```bash
php artisan translate:import-attributes
php artisan translate:import-theme --theme=demo
php artisan translate:import-attachments
php artisan translate:import-mail
```

Translated CMS page view bag keys (`localeTitle`, `localeSeo_*`, ...) need no migration; the core reads them.

## Extending SEO fields

```php
Event::listen('seo.extendSeoFields', function (array &$fields) {
    $fields['meta_author'] = ['label' => 'Author', 'tab' => 'renatio.seomanager::lang.tab.meta'];
});
```

Add the matching column to every table of a `SeoModel` model (for Tailor, to the blueprint, then `tailor:migrate`).
The field appears in model, static page and CMS page forms (except field types the page settings cannot show), is
inherited like the built-in fields and is available as `{{ seoTag.meta_author }}` in an overridden
`seotags/default.htm` partial. `seo.extendOgFields` works the same for the Open Graph tab. Both events fire on every
page render, so keep listeners free of queries and backend-only calls.

## SEO Health

**Settings > SEO > SEO Health**, its dashboard widget and `seo:doctor` report what fails silently. `seo:doctor` exits
non-zero only on a failed check, so it fits a deploy or CI step.

| Check | Catches |
|---|---|
| Site address (APP_URL) | `APP_URL` not matching the backend host or scheme |
| Settings records | a site with more than one settings record (with a button removing the unused ones) |
| Static sitemap file / Static llms.txt file | files in the web root that shadow the plugin (with a delete button) |
| Pages with URL parameters | pages the sitemap cannot fill |
| seoTags component | a theme without the component |
| SEO columns / Tailor blueprint columns | tables missing SEO columns (with fix buttons) |
| Organization data / Default Open Graph image | missing site name, deleted logo, no default image |
| Schemas from Tailor fields | broken `seoSchema` mappings |
| RainLab.Translate data | translations only RainLab.Translate holds |
| robots.txt in the project root | a root copy the web server no longer serves from `public` |
| Sitemap reference in robots.txt | no `Sitemap:` line |

The page also has the **Sitemap** status of every site, the **Structured data** preview, **Redirects**, the **404 log**,
and three audits run on a button or from the console, kept until the next run, settings save or `cache:clear`:

| Tab | Command | Reports |
|---|---|---|
| Meta titles | `seo:titles` | missing, under 30 or over 60 characters with prefix and suffix, or reused |
| Meta descriptions | `seo:descriptions` | missing, repeating the title, under 50 or over 160 characters, or reused |
| Content scores | `seo:analyze` | the content analysis score of every static page, entry and record, weakest first |

The audits cover every enabled site, compare within one site and locale, skip `noindex`, drafts and Twig expressions,
and link each row to its edit form. Name the form of your own models:

```php
Event::listen('seo.audit.editUrl', function ($record, &$url) {
    if ($record instanceof Product) {
        $url = Backend::url('acme/shop/products/update/' . $record->getKey());
    }
});
```

Add a diagnostics check: extend `Renatio\SeoManager\Classes\Diagnostics\Check`, implement `key()` and `run()` returning
`ok()`, `warning()` or `error()`, label it under `diagnostics.<key>` in your `app/lang` override, and register an
instance:

```php
Event::listen('seo.registerDiagnosticChecks', function (array &$checks) {
    $checks[] = new \Acme\Blog\Classes\FeedCheck;
});
```

## Console commands

| Command | Options | Does |
|---|---|---|
| `seo:doctor` | `--locale=` | runs the SEO Health checks; fails only on a failed check; the last line counts failed, warned and passed checks |
| `seo:titles` | `--locale=`, `--max-rows=` | meta title audit; fails when a table could not be read |
| `seo:descriptions` | `--locale=`, `--max-rows=` | meta description audit; fails when a table could not be read |
| `seo:analyze` | `--band=poor\|fair\|good`, `--limit=50`, `--fail-on=`, `--max-rows=`, `--locale=` | content scores; `--fail-on=poor` fails on any Poor row, `fair` on Fair or worse; unreadable rows or a pass cut short at `--max-rows` (20 000 by default) fail too |
| `seo:sitemap` | `--clear` | rebuilds the sitemap of every enabled site, naming the sites skipped because it is off |
| `seo:llms` | `--clear` | rebuilds llms.txt and llms-full.txt, naming the sites skipped because both are off |
| `seo:indexnow` | `{url}`, `--all`, `--scheduled` | submits to IndexNow (exit codes under IndexNow) |
| `seo:migrate-tables` | `--table=`, `--force` | adds the missing SEO columns to `SeoModel` tables after a confirmation, which a `--table` no `SeoModel` model uses asks for too |
| `seo:import-cms` / `seo:import-static` | `--dry-run`, `--force` | imports SEO values from CMS pages / static pages; `--dry-run` lists the theme files it would rewrite |
| `seo:import-blog` | | imports SEO values from blog posts and categories |
| `seo:patch 3.0` | `--force` | moves data from `renatio_seomanager_seo_tags` (upgrade to 3.0) |

`--force` skips a confirmation; a declined one, or one a non-interactive run cannot ask, exits `1`, so deploy scripts
pass `--force`. `--locale` translates the labels only, the audits read the content in the site's language.

```bash
php artisan seo:analyze --band=poor --limit=200
php artisan seo:analyze --fail-on=poor --max-rows=50000
```

## Configuration files

Create `config/renatio/seomanager/<file>.php` in your project with only the keys you change.

| File | Keys |
|---|---|
| `htaccess.php` | `tab`: `true`, `false` or `null` (detect) |
| `migrate.php` | `add_seo_columns`: add SEO columns on `october:migrate` |
| `canonical.php` | `query_params`: the page-number parameters the canonical keeps (`['page']`) |
| `indexnow.php` | `endpoint`, `debounce`, `bulk_threshold`, `timeout`, `frontend_fields` |
| `record_image.php` | `sources`, `search_attachments` |
| `content_text.php` | `sources` |
| `og_locales.php` | language → Open Graph locale |
| `social_icons.php` | service → `name`, `viewBox`, `path` (prefer `seo.extendSocialIcons`) |
| `ai_crawlers.php` | the crawler list (prefer `seo.extendAiCrawlers`) |

## Events

| Event | Arguments | Use |
|---|---|---|
| `seo.beforeComponentRender` | `$component, $page` | assign `$component->seoTag` |
| `seo.extendSeoFields` / `seo.extendOgFields` | `&$fields` | add or change form fields |
| `seo.extendOgTypes` | `&$types` | add OG types |
| `seo.extendOgLocales` | `&$locales` | change the language → locale map |
| `seo.extendImageSources` | `&$sources, $model` | image attribute names per model |
| `seo.extendJsonLd` | `&$schemas, $seoTag, $settings, $isMissingAddress` | add or change schemas |
| `seo.extendSchemaMapping` | `&$mappings, $record` | schema mappings without a blueprint |
| `seo.extendSocialIcons` / `seo.extendSocialProfiles` | `&$icons` / `&$profiles, $settings` | icons and profiles |
| `seo.alternateSites` | `&$sites` | sites of the hreflang set |
| `seo.extendHreflang` | `&$links, $controller` | hreflang links of a page |
| `seo.extendSitemap` | `&$sitemap` | add sitemap URLs |
| `seo.blogPostModel` | | return a `Post` subclass |
| `seo.extendLlmsTxt` / `seo.extendLlmsContent` | `&$sections` / `&$content, $descriptor` | llms documents |
| `seo.extendAiCrawlers` | `&$crawlers` | add AI crawlers |
| `seo.resolveContentText` | `&$text, $record, $posted` | body text for the analysis |
| `seo.extendAnalysisChecks` | `&$checks` | add analysis checks |
| `seo.registerDiagnosticChecks` | `&$checks` | add SEO Health checks |
| `seo.audit.editUrl` | `$record, &$url` | edit link in the audits |
