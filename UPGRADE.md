# Upgrade guide

Versions not listed here need no action. Back up the database before upgrading.

## Upgrading To 7.16.0

Everything since 7.3.1 ships at once. Back up the database, run `php artisan october:migrate` and
rebuild the sitemap.

**Settings are stored per site.** The migration copies the one record to every site; the sites diverge only once one
of them is edited. The robots.txt and .htaccess text is no longer kept in the record.

**Editing .htaccess is off by default.** The .htaccess tab shows a switch in place of the editor. The **Change
.htaccess** permission, and the new **Add the missing SEO columns** and **Delete the static sitemap file**
permissions, are held back from every built-in role but Developer; give editors who need them a custom role.

**`october:migrate` adds the missing SEO columns** to every table of a model carrying the `SeoModel` behavior. A deploy
script running `seo:migrate-tables --force` can drop that line. Set `add_seo_columns` to `false` in
`config/renatio/seomanager/migrate.php` to keep it out of other plugins' tables.

**The `seoTags` component renders from a copy of the record.** A listener comparing `$component->seoTag` with the
controller variable by identity has to compare keys instead. `seo.extendSeoFields` and `seo.extendOgFields` now fire
on every public request, so keep those listeners cheap. A field added through `seo.extendSeoFields` also reaches the
SEO settings of CMS pages.

**The tags change on a few pages.** A record or a CMS page without an SEO title is titled after its own title instead
of an empty tag. `og:locale` carries a territory (`de_DE`, overridable in `config/renatio/seomanager/og_locales.php`).
Error responses get `noindex` and no canonical, and a 404 or 410 no Open Graph tags. The Organization schema and the
article publisher carry the site URL; drop a `seo.extendJsonLd` listener that added it.

**The sitemap grows.** CMS pages with a URL parameter are filled from October's page finder, records that redirect or
hidden static pages are left out, and a map over 50 000 URLs or 50 MB is split into numbered files behind an index.

**The JSON-LD tab is now the Organization tab**, with the identity, contact data, address and social profile rows,
and a new `socialProfiles` component draws the profiles on the page.

**`og_type` is a dropdown** of the types in `Renatio\SeoManager\Classes\OgTypes` plus whatever the record stores;
extend it through `seo.extendOgTypes`. The Translate SEO popup of a CMS page offers only the fields stored per locale.

**New validation on save.** **Canonical URL** takes a path from the root or an absolute http(s) address, **Meta Robots
Advanced** only known directives, and the string fields are capped at 255 characters. A record holding another value
keeps rendering until it is edited.

**`seo:doctor` exits non-zero on a failed check**, including a default Open Graph image gone from the media library.
It and `seo:descriptions` report in the backend locale; pass `--locale` for another.

## Upgrading To 1.1.0

Plugin requires October CMS build 420+ with Laravel 5.5 and PHP >=7.0.

## Upgrading To 1.2.0

The `seo.beforeComponentRender` event now receives `$component` as the first parameter and `$page` as the second.
Access the SEO tag through `$component->seoTag`.

## Upgrading To 2.0.1

Plugin requires October CMS 2.x with Laravel 6 and PHP >=7.2.9.

## Upgrading To 3.0.0

The `seo_tag` relation on models implementing the SeoModel behavior is gone; SEO fields are now columns on the
model's own table. Run:

```
php artisan seo:migrate-tables
php artisan seo:patch 3.0
```

The first adds the columns to every table of a model implementing the behavior, the second moves the data over from
`renatio_seomanager_seo_tags`.

Listeners of `seo.beforeComponentRender` now assign the model itself instead of its `seo_tag` relation:

```php
$component->seoTag = $page->controller->vars['product'];
```

## Upgrading To 3.1.0

Plugin requires October CMS 2.1 or higher.

## Upgrading To 4.0.0

Plugin requires October CMS 3.0 or higher, Laravel 9.0 or higher and PHP >=8.0.

## Upgrading To 5.0.0

Plugin requires October CMS 3.1 or higher.

## Upgrading To 6.0.1

Plugin requires October CMS 4.0 or higher.

## Upgrading To 7.0.0

Plugin requires October CMS 4.1 or higher.

JSON-LD output is enabled by default after updating. Disable it in the settings if you do not want it.

The `seo.extendSeoFields`, `seo.extendOgFields` and `seo.extendJsonLd` events now pass data by reference. Listeners
modify the array directly instead of returning it:

```php
Event::listen('seo.extendSeoFields', function (&$fields) {
    $fields['my_field'] = [...];
});
```

## Upgrading To 7.1.0

The XML sitemap is disabled by default. Enable it in **Settings > SEO Configuration > Sitemap** to serve
`/sitemap.xml`.

## Upgrading To 7.1.1

If you have overridden the `seotags/default.htm` component partial, update the JSON-LD loop — each `schema` is
already a JSON string:

```twig
{% for schema in jsonLdSchemas %}
<script type="application/ld+json">{{ schema|raw }}</script>
{% endfor %}
```

## Upgrading To 7.2.3

The plugin requires `october/rain` `^4.0` again instead of `^4.1`; nothing in it depends on October 4.1.

The **Common meta tags** field is now guarded by its own **Change common meta tags** permission. Grant it to the roles
that should keep editing the field. Super users are unaffected.

Meta, `og:` and `twitter:` descriptions are now escaped, so HTML written into those fields no longer renders as
markup.

Cached sitemaps moved from `storage/app` to `storage/app/seomanager` and are rebuilt on the next request. The old
`storage/app/sitemap-*.xml` files are no longer read by anything and can be deleted by hand.

## Upgrading To 7.3.0

A blank SEO field on a Tailor entry or a `SeoModel` record no longer produces an empty tag: it falls back to the same
field of the CMS page rendering it. Pages that relied on an entry blanking out a value set in the page's SEO tab now
render the page value — clear it from the page as well if that was intended. The canonical URL, the 301 redirect and
`og:type` are excluded from the fallback. The values are written onto the record while the `seoTags` component renders,
after the `seo.beforeComponentRender` event. The sitemap applies the same rule, so an entry with a blank `robot_index`
on a `noindex` page is no longer listed.

## Upgrading To 7.3.1

Projects locked to Guzzle 8 could not install versions 7.1.0 to 7.3.0, as `spatie/laravel-sitemap` 7.x (and 8.0.0)
requires Guzzle 7; the plugin now also accepts `spatie/laravel-sitemap` 8.0.1 or newer, which needs PHP 8.4 or newer.
Run `composer update renatio/seomanager-plugin -W` to let Composer pick it.
