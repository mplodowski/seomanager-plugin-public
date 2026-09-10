# Renatio SEO Manager Plugin

Meta tags, Open Graph, JSON-LD, hreflang and XML sitemaps for [October CMS](https://octobercms.com) — ready for Google
and AI search.

**Demo URL:** https://october-demo.renatio.com/backend/backend/auth/signin  
**Login:** seomanager  
**Password:** seomanager

Supports CMS Pages, RainLab.Pages, RainLab.Blog and October CMS Tailor out of the box, and attaches to any October
CMS model with one line of code.

## Features

* Support for CMS Pages, RainLab.Pages, RainLab.Blog
* Import default values from CMS Pages, RainLab.Pages, RainLab.Blog
* Open Graph support
* Live Google snippet and Facebook / X share card preview in the backend forms
* Robots meta configuration
* Canonical URLs
* 301 Redirects
* JSON-LD structured data (Schema.org Organization or LocalBusiness, WebSite, BreadcrumbList, Article/BlogPosting)
* Social profile links drawn on the page with brand icons, from the same addresses the structured data uses
* Hreflang alternate language tags for multisite
* XML Sitemap generation with multisite and hreflang support
* Easily edit robots.txt in CMS Settings, plus .htaccess once you enable it
* Easy integration with other plugins
* Fully compatible with RainLab.Translate for multi-lingual support
* Fully support for October CMS Tailor

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

## Requirements

- October CMS 4.x
- PHP 8.2 or newer, with the `dom` extension
- `spatie/laravel-sitemap` 7.2 or 8.x, installed by Composer with the plugin

## Usage

After installation CMS pages, static pages, blog posts and categories get additional SEO and Open Graph tabs.

To output the tags, place the `seoTags` component in the head of your layout:

```
[seoTags]
==
<!DOCTYPE html>
<html>
    <head>
        {% component 'seoTags' %}
    </head>
```

### Where the tags come from

When the page renders a Tailor entry with the SEO mixins, or a model using the `SeoModel` behavior (blog posts and
categories), that record is the source of the SEO tags. A record without an **SEO Title** is titled after its own
`title` or `name`. Any other field the record leaves blank falls back to the same field
of the CMS page, so a page-level description or `robot_index` still applies until the entry overrides it. The
canonical URL, the 301 redirect and `og:type` are never inherited, as they describe one URL. A page without such a
record uses its own SEO tab. The sitemap follows the same rule for `noindex`.

## October CMS Tailor

Add the two mixins to the `fields` section of your blueprint, then save and migrate the blueprint (or run
`php artisan october:migrate`):

```
_seo_meta_fields:
    type: mixin
    source: Renatio\SeoManager\MetaFields

_seo_og_fields:
    type: mixin
    source: Renatio\SeoManager\OgFields
```

## Settings

Go to **Settings > SEO Configuration**. The settings cover:

* the site name (used for Open Graph and structured data) and an SEO title prefix/suffix;
* the locale the site publishes in `hreflang` and `og:locale`, e.g. `de-CH`, for a site whose October locale names
  the language alone - see **The locale of the share card** and **Hreflang Tags**;
* common meta tags added to every page, e.g. `<meta name="author" content="Renatio">`;
* Open Graph output: Facebook Application ID, a default image for pages without their own, and the Twitter/X
  site handle. The component also outputs `og:locale` for the active site, `og:locale:alternate` for
  the other sites, and defaults `og:type` to `article` for entries with a publish date and `website` otherwise;
* everything structured data needs, on the **Organization** tab: the JSON-LD switch itself and, under it, the
  Schema.org identity, the logo, the contact data, the postal address, the founding date and, for a local business,
  the coordinates and the opening hours;
* the **social profiles** of the organization, also on the Organization tab and outside the JSON-LD switch: one row
  per profile, carrying the address and the service it belongs to. They feed `sameAs` in the structured data and the
  `socialProfiles` component below;
* robots.txt and .htaccess, read from and written to the web root (`public` when it exists, otherwise the project
  root). The files are only written when their content changes, and .htaccess is never overwritten with empty
  content.

> Common meta tags are placed in the page head exactly as typed, without escaping, so anyone who can edit the field
> can put scripts on every page. The field is guarded by its own **Change common meta tags** permission.

> Editing .htaccess may break your site, so it is off by default. Turn it on with the switch on the .htaccess tab
> when you need it. Both the switch and the editor are guarded by the **Change .htaccess** permission.

### Permissions

The settings page opens with **Manage SEO settings**, which the built-in Developer and Publisher roles hold. The
fields whose reach goes beyond one site are guarded on top of that, and a field the administrator may not change is
left out of the form rather than shown read-only:

| Permission | Guards | Given by default to |
| --- | --- | --- |
| Manage SEO settings | the settings page and every field not listed below | Developer, Publisher |
| View SEO Health and run its checks | the SEO Health page, its dashboard widget and the sitemap rebuild | Developer, Publisher |
| Change robots.txt and the AI crawler policy | the robots.txt editor, its reset button and the AI crawler policy | Developer |
| Change the allowed redirect hosts | the list of external hosts a 301 redirect may point to | Developer |
| Change common meta tags | the raw HTML placed in the head of every page | Developer |
| Change .htaccess file | the .htaccess switch and editor | Developer |
| Add the missing SEO columns | the button on the SEO Health page that alters tables of other plugins | Developer |
| Delete the static sitemap file | the button on the SEO Health page that deletes a file from the web root | Developer |

A superuser holds every permission. October locks the permissions of its built-in roles, so an editor who needs one
of the Developer-only permissions has to sit on a custom role with it ticked in **Settings > Administrators > Roles**.

### Multisite

With [multisite](https://docs.octobercms.com/4.x/cms/resources/multisite.html) the settings are stored per site: pick
the site in the backend site switcher and the page shows and saves that site's own values. Site name, title
prefix/suffix, common meta tags, the Open Graph, Organization and JSON-LD fields and the sitemap switch are all
separate per site, so a German and an English site can carry different names, addresses, default images and structured
data.

Four things stay shared by the whole installation, because they are backed by a single file or a single scheduled task,
and the settings page labels them as such:

* **robots.txt** and **.htaccess**, including the AI crawler policy and the .htaccess switch — there is one public
  directory per installation;
* the **allowed redirect hosts**, a security list that is easier to audit in one place;
* the **sitemap rebuild schedule** — one scheduler serves the whole installation.

Upgrading an existing installation gives every site a copy of the settings it had before, so nothing changes until you
edit one of the sites. A site added later starts from the defaults, with the shared fields taken from the sites that
already exist, and serves a sitemap if any of them does.

## AI crawlers

The **robots.txt** tab takes a single **AI crawler policy**: *Leave alone*, *Allow AI search, block training*,
*Allow all*, *Block all* or *Custom per crawler*. Only the last one shows the full list, as a table of each crawler
with its vendor and what it is for:

- **search** crawlers (`OAI-SearchBot`, `ChatGPT-User`, `Claude-SearchBot`, `Claude-User`, `Bravebot`,
  `PerplexityBot`, `Amazonbot`) fetch a page to answer a visitor's question and can send traffic back;
- **training** crawlers (`GPTBot`, `ClaudeBot`, `Google-Extended`, `Applebot-Extended`, `CCBot`, `Bytespider`)
  collect content for model training and send nothing back.

That split is what *Allow AI search, block training* acts on.

The policy is written into a marked block; everything outside it is left untouched, and saving again replaces the
block. Crawlers left alone are not written at all, so a rule you wrote by hand stays in charge. The settings page
warns when your own rules already declare a `User-agent:` group for a crawler the policy covers.

```
# BEGIN SEO Manager AI crawlers
User-agent: GPTBot
Disallow: /

User-agent: OAI-SearchBot
Allow: /
# END SEO Manager AI crawlers
```

To add your own crawlers, listen to `seo.extendAiCrawlers`. The `purpose` key decides how the *Allow AI search, block
training* preset treats the crawler:

```php
Event::listen('seo.extendAiCrawlers', function (&$crawlers) {
    $crawlers[] = ['agent' => 'AcmeBot', 'vendor' => 'Acme', 'purpose' => 'training'];
});
```

## SEO fields

* On a CMS page, the **Meta** section October itself offers (meta title, description, image, type and robot
  directives) fills in whichever of these fields the SEO popup leaves blank; a value in the SEO popup always wins.
* **SEO Title** - the document title, shown in search results and when sharing.
  [Read more](https://moz.com/learn/seo/title-tag)
* **SEO Description** - the meta description shown as the snippet in search results.
  [Read more](https://moz.com/learn/seo/meta-description)
* **Meta Robots** - the robots meta tag, used together with robots.txt. Index and follow are chosen with the radio
  buttons; **Meta Robots Advanced** takes the remaining directives, comma separated, such as `noarchive`,
  `nosnippet`, `max-snippet:150` or `max-image-preview:large`, and refuses one the tag does not know.
  [Read more](https://yoast.com/robots-meta-tags/)
* **Canonical URL** - the dominant URL when several point at the same content: a path from the root or an absolute
  http(s) address, on any host.
  [Read more](https://moz.com/learn/seo/canonicalization)
* **Meta Keywords** - keywords relevant to the page. Google and Bing ignore them, so the field sits last.
* **301 Redirect** - when filled, every request of that page is redirected, as long as the `seoTags` component is
  present in the page or its layout. The target must be a relative path (`/new-page`) or an absolute URL on an
  allowed host: the hosts of the configured sites plus **Settings > SEO Configuration > Allowed redirect hosts**.
  Other targets are rejected on save and ignored at runtime, and no redirect happens when the target equals the current URL or on error pages.

### Error responses

A response with a status of 400 or above gets `noindex, nofollow`, whether it comes from the theme's `404` and
`error` pages or from a page that sets its own status code, such as a catch-all route answering 404 for an unknown
slug. **Meta Robots** is not asked here: the field defaults to `index` everywhere it is defined, so a stored
`index` cannot be told apart from one nobody chose. Maintenance mode is the exception and keeps every tag - it
answers 503 from an ordinary page, and "come back later" is not "this address does not exist".

When the address itself is not there - the theme error pages, or a status of 404 or 410 - the tags that name the
page go too:

* no canonical link and no alternate language links, since a missing address is nobody's canonical and has no
  translations. A **Canonical URL** filled in on the page still wins and is rendered as given, which is the way to
  point every error response at a page that does exist;
* no Open Graph and Twitter tags, because there is nothing to share. Facebook reads `og:url` the way a crawler
  reads the canonical, so leaving it would undo the line above;
* JSON-LD keeps only the `Organization` and `WebSite` schemas, which describe the site rather than the address.
  `BreadcrumbList` and `Article` are built from the requested path and are left out.

A gated or broken page - 401, 403, 500 - answers from an address that does exist, so it keeps its canonical, its
alternates and its share card. Only the `noindex` applies.

## Open Graph fields

* **OG Title**, **OG Description**, **OG Type** and **OG Image**, as defined by the
  [Open Graph Protocol](http://ogp.me/).

**OG Type** offers the object types a site is likely to share; "Automatic" renders `article` for a record with a publish
date and `website` otherwise. Add a type of your own with the `seo.extendOgTypes` event:

```php
Event::listen('seo.extendOgTypes', function (array &$types) {
    $types[] = 'music.playlist';
});
```

> More fields can be added on request.

### The locale of the share card

`og:locale` is built from the locale of the active site, and `og:locale:alternate` from the other sites of the group.
Open Graph reads them as `language_TERRITORY` and ignores a bare `de`, while `hreflang` wants exactly that bare code,
so a site whose locale names a language alone is given the territory the language is usually spoken in - `de` becomes
`de_DE`, `pl` becomes `pl_PL`. A locale that already names a territory keeps it, whatever the separator: `pt-br`
becomes `pt_BR`.

Two sites serving one language to two countries carry the same locale in October, because the locale names the
language their translations are written in. Name the region on the **General** tab of the SEO settings of each site -
`de-DE` on one, `de-CH` on the other - and that value is what the site publishes: `de_CH` here, `de-CH` in
`hreflang` and in the sitemap alternates.

Override the map in `config/renatio/seomanager/og_locales.php` of your project, or change a single language:

```php
Event::listen('seo.extendOgLocales', function (array &$locales) {
    $locales['de'] = 'de_AT';
});
```

### Where the OG image comes from

A record with no **OG Image** of its own falls back to the image it already shows on the page, before the default
image from the settings is used. The plugin checks, in order, `banner`, `featured_images`, `image`, `images`,
`photo`, `photos`, `cover` and `thumbnail` - as an attribute (Media Library path) or as an `attachOne` /
`attachMany` relation, taking the first file of a multiple one. When no name matches, the file attachments the
model declares are searched in the order of the declaration, so an image under a name of your own is found
without any configuration.

Only a public image counts - `jpg`, `jpeg`, `png`, `gif`, `webp` or `avif` - so a protected attachment, a PDF or
an SVG is skipped and the default image from the settings is kept. The same image is used for JSON-LD. An image
taken from an attachment carries no `og:image:width` and `og:image:height`, since those are measured only on the
Media Library disk.

The list decides which image wins when a model has several, so name yours there rather than relying on the search
over the attachments, which follows the order the model happens to declare them in. Override the list in
`config/renatio/seomanager/record_image.php` of your project - the same file turns the search off with
`'search_attachments' => false` - or add a name for a single model:

```php
Event::listen('seo.extendImageSources', function (array &$sources, Model $model) {
    if ($model instanceof Album) {
        $sources[] = 'cover_photo';
    }
});
```

## Preview in the backend

The **SEO** tab opens with the Google result the record would produce, and the **Open Graph** tab with the Facebook
link box and the X (Twitter) card, drawn the way each network renders a share, with or without an image. Both redraw while you type, on the models carrying the `SeoModel` behavior, on
Tailor entries and on the static pages of RainLab.Pages alike.

Everything the preview needs is worked out once while the form is built, by the same classes the page render goes
through, and the script only substitutes what you type. A field you leave empty shows in grey italics what the
render would really use instead: the title of the record between the prefix and suffix from the settings for an
empty **SEO Title**, the **SEO Title** in place of an empty **OG Title**, the **SEO Description** in place of an empty **OG Description**,
and the image the record shows on the page - or the default image from the settings - in place of an empty
**OG Image**.

The title and the description are measured in pixels rather than counted in characters, against the 580 and 920
pixels a desktop result fits, because that is where the search engines truncate. The character counters on the
fields themselves are unchanged.

The snippet names the address the record is served from, as the sitemap would list it: the page holding the
blog post, the category or the Tailor section is found in the theme, its pattern is filled with the record's own
values and the slug follows the **Slug** field while you type. A record no page is known to serve is shown right
under the site. A **Canonical URL** replaces the address once it is filled in. The favicon comes from the theme
(`assets/images/favicon.*`, `assets/img/favicon.*` or `assets/favicon.*`) or from the web root, and the grey circle
stays when there is none.

Two things the preview cannot know. What a blank field inherits from the page's own SEO settings on render: the
preview shows the title composed from the plugin settings where the render would use the page's title, and nothing
at all where the render would inherit the page's description. And the record's own state: one that has never been
saved holds no relation to read, so its preview falls straight through to the default image from the settings.

## JSON-LD Structured Data

The `seoTags` component outputs [JSON-LD](https://json-ld.org/) structured data (Schema.org) as
`<script type="application/ld+json">` blocks. It lets Google show rich results (breadcrumbs, article cards, the site
name and logo) and lets AI search engines understand who publishes the site and what each page is about.

JSON-LD is enabled by default. The **Organization** tab in the settings holds the switch and every field the schemas
read and warns about what is missing. The **SEO Health** page links to the Schema.org validator and the Google Rich
Results Test and previews the JSON-LD generated for the home page.

### Generated schemas

* **Organization** - always included; uses the site name, the site base URL and everything filled in on the
  Organization settings tab. Pick one of the LocalBusiness types there and the schema is emitted under that type
  instead, with the geo coordinates and the `openingHoursSpecification` Schema.org allows only on a local business.
  Every field left empty is left out of the schema rather than published blank.
* **WebSite** - always included; uses the site name and the current site base URL.
* **BreadcrumbList** - every page except the homepage, built from the URL path with the page title as the last item.
  Multisite route prefixes are excluded.
* **Article / BlogPosting** - pages whose model has `title` and `published_at` attributes. Tailor entries produce
  `BlogPosting`, other models `Article`.

| Schema property | Resolution order                                                 |
|-----------------|------------------------------------------------------------------|
| headline        | `meta_title` → `title` (max 110 chars)                          |
| description     | `meta_description` → `excerpt` → `content` (first 200 chars)    |
| image           | `og_image` → the file relations listed under Open Graph fields  |
| author          | `user` relation → `author` string → `author` relation           |
| publisher       | Organization from settings, with its name, url and logo          |
| datePublished   | `published_at`                                                   |
| dateModified    | `updated_at` → `published_at`                                   |

### Extending JSON-LD

```php
Event::listen('seo.extendJsonLd', function (array &$schemas, mixed $seoTag, Settings $settings, bool $isMissingAddress) {
    $schemas[] = [
        '@context' => 'https://schema.org',
        '@type' => 'FAQPage',
        'mainEntity' => [ /* ... */ ],
    ];
});
```

## Social profiles

The profiles entered on the **Organization** settings tab are the addresses the `Organization` schema publishes as
`sameAs`. The `socialProfiles` component draws the same rows on the page, so the footer and the structured data never
drift apart:

```
[socialProfiles]
==
{% component 'socialProfiles' %}
```

Every link is drawn with the brand icon of its service as inline SVG, needs no icon font and carries an `aria-label`
with the name of the service, because an icon alone says nothing to a screen reader. The rows are drawn in the order
they are entered, and with multisite each site draws its own.

| Property   | Default          | Description                                                                     |
|------------|------------------|---------------------------------------------------------------------------------|
| `only`     |                  | Draw only these services, comma separated, e.g. `facebook,instagram`             |
| `except`   |                  | Leave these services out, comma separated                                        |
| `target`   | `_blank`         | The `target` of every link; empty opens the profile in the same tab              |
| `class`    | `social-profiles`| The class of the list and the stem of the element classes                        |
| `iconSize` | `24`             | The width and the height of the icon, in pixels                                  |
| `iconClass`|                  | Draw an icon font instead of the SVG, e.g. `fab fa-:icon`                        |

`:icon` in `iconClass` is replaced with the service, so `fab fa-:icon` renders `<i class="fab fa-facebook">`.

### Overriding the markup

Copy the default partial to `themes/<theme>/partials/socialProfiles/default.htm` and October renders yours instead.
Each entry carries `url`, `icon` (the service), `name` (what a screen reader reads), `path` and `viewBox` (the icon
itself):

```
{% for profile in __SELF__.profiles %}
    <a href="{{ profile.url }}" aria-label="{{ profile.name }}" target="_blank" rel="noopener noreferrer">
        <svg viewBox="{{ profile.viewBox }}" width="20" height="20" fill="currentColor" aria-hidden="true">
            <path d="{{ profile.path }}"></path>
        </svg>
    </a>
{% endfor %}
```

### Extending the icons and the profiles

The icons live in `config/social_icons.php` and are keyed by the value the settings store. Add your own, or replace a
path, with the `seo.extendSocialIcons` event:

```php
Event::listen('seo.extendSocialIcons', function (array &$icons) {
    $icons['goldenline'] = [
        'name' => 'GoldenLine',
        'viewBox' => '0 0 24 24',
        'path' => 'M12 0C5.4 0 ...',
    ];
});
```

Profiles kept somewhere else - in a Tailor blueprint, in another plugin's settings - are appended with
`seo.extendSocialProfiles`. A row only needs `url` and `icon`; the icon itself is filled in from the icon set, and so
is the name a screen reader reads unless the row carries a `name` of its own:

```php
Event::listen('seo.extendSocialProfiles', function (array &$profiles, Settings $settings) {
    $profiles[] = ['url' => 'https://www.youtube.com/@acme', 'icon' => 'youtube'];
});
```

Icons come from [Simple Icons](https://simpleicons.org) (CC0), except LinkedIn and the generic globe, which come from
[Bootstrap Icons](https://icons.getbootstrap.com) (MIT).

## Hreflang Tags

When [multisite](https://docs.octobercms.com/4.x/cms/resources/multisite.html) is active with two or more sites in the
same site group, the `seoTags` component outputs `<link rel="alternate" hreflang="...">` tags for each enabled site,
plus an `x-default` entry pointing to the primary site. No configuration is needed.

The links carry no query string, so they match the `canonical` and `og:url` tags, which the component builds from the
request path alone. That also drops a query string contributed by RainLab.Translate's `cms.sitePicker.overrideQuery`
event, which is its documented way of translating query parameters between sites. To put parameters back, rewrite the
links through `seo.extendHreflang` — or, for the sitemap, through `seo.extendSitemap`.

The same set of sites drives the sitemap alternates, so the two never disagree. Two rules decide who is in it:

- **Site groups are all or nothing.** October returns every enabled site when no group is configured anywhere, so an
  installation without groups cross-links all of its sites, while a site alone in its own group gets no alternates at
  all. Assign groups to every site or to none — a half-grouped installation is worse than either.
- **Every site of a group needs its own language.** A site with no locale of its own reports the application locale, so
  two of them claim the same language. One `hreflang` code on two URLs contradicts itself and search engines discard
  the set, so the sites claiming that language drop out and the correctly configured ones keep their annotations. A
  site that dropped out publishes nothing at all rather than a set without a link to itself. The Sitemap settings tab
  names the language responsible. Two sites that really share a language tell themselves apart by the locale on the
  **General** tab of their SEO settings, `de-DE` and `de-CH`.

### Extending Hreflang

```php
Event::listen('seo.extendHreflang', function (array &$links, Controller $controller) {
    $links[] = [
        'locale' => 'fr',
        'url' => 'https://example.com/fr/page',
    ];
});
```

## XML Sitemap

The plugin serves a sitemap at `/sitemap.xml` that includes:

- CMS pages, RainLab.Pages static pages, RainLab.Blog posts and categories, and Tailor entries that have a CMS page
  with a `section` component;
- the records any other plugin serves on a page with URL parameters, read through October's page finder - see
  **Pages with URL parameters** below;
- `lastmod` from `updated_at` timestamps;
- one sitemap per site in multisite, with translated URLs and `xhtml:link` hreflang alternates for every entry, post,
  category and static page - each alternate built from the record that belongs to the other site, so a translated slug
  points at the translated URL. Blog posts kept per site - a `Post` subclass with October's `Multisite` trait - are
  listed for their own site only, with the alternates taken from the siblings sharing their `site_root_id`.

Alternates cover the same sites as the `seoTags` hreflang tags — see the two rules under **Hreflang Tags** — except
that a per-site post missing from a site gets no alternate for it in the sitemap.

Hidden pages, anything with `noindex` and anything with a 301 redirect URL are excluded. The sitemap is cached in `storage/app/seomanager` and rebuilt
by the scheduler, so October's cron entry has to be in place:

With the sitemap switched off, or under a prefix no site answers to, the address is handed to the CMS as if the plugin
had no route there, so a theme page with `url = "/sitemap.xml"` can serve it instead.

```bash
* * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1
```

Enable it in **Settings > SEO Configuration > Sitemap**. **Settings > SEO > SEO Health** lists the sitemap URL of
every site with its cache status, offers a preview link and a **Rebuild sitemap** button for publishing a change right
away, and warns when robots.txt has no `Sitemap:` line.

**Rebuild** picks how often the scheduler runs: every hour, every day at a chosen hour, every week on Sunday, or never.
A cached sitemap older than 48 hours — two weeks when the rebuild is weekly — is rebuilt inside the request that asks
for it, so a stopped scheduler cannot freeze the map for good. Pick *Never* to leave the map to that fallback alone.

The **Rebuild** button hands the work to a queue worker whenever the queue driver is anything but `sync`, which keeps
a large sitemap from hitting the execution time limit; on `sync` it rebuilds inside the request. The scheduled
rebuild runs outside the request either way.

Give the queue connection a `retry_after` longer than a full rebuild of every site takes. A job still running when
`retry_after` elapses is reserved again by another worker, which marks the first run failed even though it finishes
and writes the map correctly. A rebuild that fails is written to the log as well as to `failed_jobs`, because the
page reports only that the job was queued.

```bash
php artisan seo:sitemap          # warm the cache of every enabled site
php artisan seo:sitemap --clear  # clear the cache
```

### Files above the protocol limits

The sitemaps.org protocol caps one file at 50 000 URLs and 50 MB, and a file over either limit is rejected whole
rather than truncated — a site large enough to cross it would otherwise have no map at all, and nothing would say so.

A map that stays within the limits is written as one document, exactly as before. One that outgrows them is written
as `sitemap-1.xml`, `sitemap-2.xml` and so on, and `/sitemap.xml` answers a sitemap index naming them, which is the
address `robots.txt` and Search Console already point at. The count decides the split; a part that still weighs more
than 50 MB — hreflang alternates can make a single entry large — is halved again until it fits. Parts left behind by
a larger build are deleted, so the index never names a file from a previous run.

The **Sitemap** settings tab says how many files a map was split into, and counts the URLs across all of them.

### Pages with URL parameters

A page whose URL carries a parameter - `/products/:slug` - serves a record per address, and the sitemap cannot guess
which. It reads them from October's page finder — the `cms.pageLookup` events behind the **Page** field of a menu item
— so any plugin answering it lands in the map on its own. That covers Tailor entries, RainLab.Blog posts and
categories, and every plugin registering a type that lists all of its records.

It is the same API October's own demo theme builds `pages/sitemap.xml` from, through the `link()` Twig function -
the plugin reads it for every page instead of the entries picked in a menu.

A plugin answering only the older `pages.menuitem` events of RainLab.Pages is not read; register the same listeners
under the `cms.pageLookup` names as well, which is what the October plugins do:

```php
Event::listen(['cms.pageLookup.getTypeInfo', 'pages.menuitem.getTypeInfo'], ...);
```

The `sites` a resolver returns become `xhtml:link` alternates, narrowed to the sites the hreflang rules cover. A record
excluded with `noindex` is not recognisable through the page finder, so it is listed - the `SeoModel` records the
plugin reads first hand keep honouring it.

A page no type can address is reported on the **SEO Health** page and by `seo:doctor`, rather than dropped in
silence. Give the plugin owning the records a page finder type, or add the addresses through `seo.extendSitemap` below -
a page whose pattern one of the added addresses matches counts as covered.
A page that belongs outside the map — a paginated archive, a filter — is left out of the report once its SEO settings
say `noindex`, which is what it should be saying anyway.

### Extending Sitemap

Listen to `seo.extendSitemap` to add URLs the plugin does not discover, such as products from a custom plugin:

```php
use Acme\Shop\Models\Product;
use Cms;
use Spatie\Sitemap\Sitemap;
use Spatie\Sitemap\Tags\Url;

Event::listen('seo.extendSitemap', function (Sitemap &$sitemap) {
    foreach (Product::where('is_published', true)->get() as $product) {
        $sitemap->add(
            Url::create(Cms::pageUrl('product/show', ['slug' => $product->slug]))
                ->setLastModificationDate($product->updated_at)
        );
    }
});
```

A project serving its blog posts through a subclass of `RainLab\Blog\Models\Post` - one carrying October's `Multisite`
trait, say - keeps the translations and the site scope of that subclass, which the base model never sees. Name it
with `seo.blogPostModel` and the sitemap reads the posts through it:

```php
Event::listen('seo.blogPostModel', fn () => \Acme\Blog\Models\Post::class);
```

## Integration with models

Implement the SeoModel behavior in your model class:

```
public $implement = ['@Renatio.SeoManager.Behaviors.SeoModel'];
```

The SEO columns are added to the model's table the next time `php artisan october:migrate` runs, on every
environment, so a deploy script needs nothing more than the migration it already runs. To add them right away, this
command scans all models implementing the behavior:

```
php artisan seo:migrate-tables
```

Without those columns the behavior is dead and nothing entered in the SEO tab is stored, with no error anywhere. The
**SEO Health** page names every table this happened to and adds the columns to all of them on a button, so an
environment that has not been migrated since does not stay silently without SEO.

For the plugin to recognize the model on a page, pass it to the page view, usually in a component's `onRun()`:

```
$this->page['album'] = Album::find($id);
```

## Extending SEO fields

Listen to `seo.extendSeoFields` or `seo.extendOgFields` to modify or add fields. The events pass the array by
reference, so modify it directly instead of returning it. Fields are saved to the database, so first add the
matching columns to the table of every model that implements the behavior; for Tailor entries, add the fields to the
blueprint instead.

```
Event::listen('seo.extendSeoFields', function (&$fields) {
    // modify or add more fields
});
```

A field added this way is offered in the backend form of every model carrying the behavior, in the static page form and
in the SEO settings of a CMS page, and the `seoTags` component reads it from the page and lets an entry that leaves it
blank inherit it, so the value is on `seoTag` when the page renders. A field added through `seo.extendOgFields` follows
the Open Graph switch in the backend forms, exactly like the ones the plugin ships. A field the page settings have no
control for — a `hint`, a `repeater`, a `ruler` — is left out of them. The component partial writes out only the tags
the plugin defines; override `components/seotags/default.htm` in the theme to render `{{ seoTag.meta_author }}` as a
tag of your own.

Both events fire on the front end as well, once per rendered page, because the component builds the list of field names
from them. Keep the listeners cheap and out of anything that only exists in the backend — a database query for dropdown
options or a call to `BackendAuth` runs on public requests too.

## Access SEO Tag before rendered on page

Listen to `seo.beforeComponentRender` to assign the model whose SEO fields should be rendered. The model should
implement the SeoModel behavior.

```
Event::listen('seo.beforeComponentRender', function ($component, $page) {
    if ($page->url == '/products/:slug') {
        $component->seoTag = $page->controller->vars['product'];
    }
});
```

> Whatever a listener assigns is rendered as a meta tag value. Titles and descriptions are escaped, but nothing else is
> sanitized, so do not feed the event with content you would not put on the page yourself.

`$component->seoTag` is a copy of the record, made before the event: what the listener writes on it, and what the
component prepares afterwards — the page fallback, the title, the robots value and the Open Graph defaults — never
reaches the record the theme holds, so nothing the page renders can end up in the database. A record assigned by the
listener is copied the same way.

## Diagnostics

Several of the ways an installation can be misconfigured produce no error message at all: a static `sitemap.xml` left
in the web root is answered by the web server before the request ever reaches the plugin route, a page whose URL
carries a parameter that no plugin can list the records of falls out of the sitemap entirely, a theme that never
attaches the `seoTags` component renders no tag, a table whose model carries the `SeoModel` behavior without the SEO
columns stores nothing that is entered in the SEO tab, structured data enabled without a site name or with a logo that
has been deleted from the media library describes the site with a broken identity, Open Graph enabled without a default
image shares every page and entry that has no picture of its own without one, and a `robots.txt` that never names the
sitemap leaves crawlers to find it on their own.

**Settings > SEO > SEO Health** lists all of them, next to the sitemap status and the meta title and description
audits, and a **SEO Health** dashboard widget shows the same counts with the checks that fail. `seo:doctor` reports the
same checks on the command line, exiting with a non-zero status when one of them fails — which makes it usable as a
deploy or CI step:

```
php artisan seo:doctor
```

Listen to `seo.registerDiagnosticChecks` to add a check of your own. The event passes the array by reference, and each
entry extends `Renatio\SeoManager\Classes\Diagnostics\Check`:

```
Event::listen('seo.registerDiagnosticChecks', function (array &$checks) {
    $checks[] = new \Acme\Blog\Classes\FeedCheck;
});
```

The label of a check is read from the `renatio.seomanager::lang.diagnostics.<key>` translation key.

When the SEO columns are missing the page offers an **Add the missing SEO columns** button, which runs the same schema
change as `seo:migrate-tables` on every table it reported — so an environment that was never migrated can be repaired
without shell access. The button alters tables belonging to other plugins, so it carries its own
`renatio.seomanager.migrate_seo_columns` permission, held by the Developer role alone.

A static sitemap file found in the web root can be deleted from the same page with the **Delete the static sitemap
file** button, which removes every file and symlink the check reported. Deleting from the web root of a production
server is guarded the same way, by the `renatio.seomanager.delete_static_sitemap` permission of the Developer role.

### Meta descriptions

A meta description nobody wrote costs more than none at all. An empty one lets the search engine quote the page
itself; a placeholder repeating the title — `Kontakt`, `Produkte`, `News` — is quoted as it stands. Since the page
fallback landed, it also travels: a record that has no description of its own inherits the one on the CMS page it is
rendered on, so a single placeholder on `/product/:slug` describes every product under it.

The **Meta descriptions** tab of the same **SEO Health** page carries a **Check again** button, and `seo:descriptions`
prints the whole list on the command line:

```
php artisan seo:descriptions
```

Both report every page, static page, Tailor entry and model record whose description is missing, only repeats a
heading the record already carries, is shorter than 50 characters or is used under more than one address. A page
whose URL carries a parameter says so on its own row: what it holds, or fails to hold, reaches every record rendered
on it.

Each row links to where the text is changed: the Editor for a CMS page, the Static Pages list, the entry form for a
Tailor entry and the post or category form for RainLab.Blog. A project holding `SeoModel` records of its own names
their address through `seo.audit.editUrl`:

```php
Event::listen('seo.audit.editUrl', function ($record, &$url) {
    if ($record instanceof Product) {
        $url = Backend::url('acme/shop/products/update/' . $record->getKey());
    }
});
```

Only what a visitor can reach is read: records set to `noindex`, drafts and entries outside their publishing window
are left out, along with the `/404`, `/error` and `/sitemap.xml` addresses. A page set to `noindex` is left out too
unless its URL carries a parameter — its description still reaches the records rendered on it, which are indexed on
their own. A page inherits the description its layout sets in a `[viewBag]` section, the same way the `seoTags`
component resolves one, so a theme-wide default is not reported as missing on every page using it.

Descriptions are compared within one audience: a site, and under RainLab.Translate a locale, so two sites or two
languages holding their own copy of the same text are not counted as a repeat. A theme file, or a model that knows
nothing about multisite, belongs to every audience and is compared against the records of each. A locale is read
only where it holds a translation of its own; without one the page falls back to the stored value, which is already
in the report.

A description written as a Twig expression is left alone. `meta_title` and `meta_description` are parsable attributes
of a CMS page, so `{{ post.summary }}` there is the documented pattern and the text is settled per record at render
time — reporting the expression itself as too short, or as a placeholder handed to every record under the page, would
be wrong on both counts.

The scan reads every record the plugin can describe, one row at a time, so it runs on request rather than on every
render of the settings page. A table it cannot read is named in the report rather than passed over, and a site whose
content outgrows what one report can hold is told that only part of it was checked — a clean bill of health produced
by a failed query, or by a scan that stopped early, is the one answer this is meant not to give.

### Meta titles

The **Meta titles** block of the same tab, and `seo:titles` on the command line, read the same content and report the
title instead: an address with no title at all, one that outgrows 60 characters once the prefix and suffix of the
settings are added — the length measured is the composed title, because that is what a result page prints — and one
used under more than one address, compared within a single site and locale the same way descriptions are.

```
php artisan seo:titles
```

A record without a meta title of its own is not reported: the page is still listed under the heading it carries, which
is what the `seoTags` component falls back to.

## Translations

The backend ships in Czech, Dutch, English, French, German, Italian, Polish, Brazilian Portuguese, Russian and
Spanish, and follows the language of the October backend user. `tests/Unit/TranslationsTest.php` compares every
locale against `lang/en/lang.php`: a key that is missing, one that no longer exists, a line whose `:placeholders`
drifted and an empty line each fail the suite, so a locale cannot silently rot behind the reference.

Names that are the same everywhere are left in English: Open Graph, JSON-LD, hreflang, Schema.org, `robots.txt`,
`.htaccess`, the artisan commands and the `seoTags` component.

## Console commands

* `seo:doctor` - report the configuration problems that fail silently; exits with a non-zero status on a failed check
* `seo:descriptions` - list the pages and records whose meta description is missing, copied from the title, too
  short or reused; both reports are written in the backend locale, `--locale=de` picks another language
* `seo:titles` - list the pages and records whose meta title is missing, longer than 60 characters with the prefix
  and suffix, or reused; takes the same `--locale=` option
* `seo:migrate-tables` - add SEO columns to the tables of all models implementing the SeoModel behavior; `--table=`
  limits it to one table, `--force` skips the confirmation (required in deploy scripts and CI)
* `seo:patch 3.0` - migrate data from `renatio_seomanager_seo_tags` to models implementing the behavior
* `seo:import-cms` - import SEO from CMS pages
* `seo:import-static` - import SEO from RainLab static pages
* `seo:import-blog` - import SEO from RainLab Blog posts and categories
* `seo:sitemap` - warm the XML sitemap cache of every enabled site; `--clear` clears it
