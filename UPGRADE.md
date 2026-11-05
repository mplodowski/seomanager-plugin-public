# Upgrade guide

Versions not listed here need no action. Back up the database before upgrading.

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

The XML sitemap is disabled by default. Enable it in **Settings > SEO > SEO Manager > Sitemap** to serve
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

## Upgrading To 8.0.0

Requires October CMS 4.4. Run `php artisan october:migrate`: with `add_seo_columns` on (the default) it also adds the
new **Focus Keyword** column to `SeoModel` tables and to Tailor tables using the SEO mixins; where it is off, run
`php artisan seo:migrate-tables --force` and `php artisan tailor:migrate`. Then rebuild the sitemap.

- From a version below 7.2.3, upgrade to 7.3.2 first.
- Settings are stored per site; the migration copies the current ones to every site.
- **Change the .htaccess file**, **Change common meta tags** and the new **Change robots.txt and the AI crawler
  policy**, **Change the allowed redirect hosts**, **Add the missing SEO columns** and **Delete the static sitemap
  file** are Developer only; the new **View SEO Health and run its checks** goes to Developer and Publisher. Tick them
  for custom roles.
- The .htaccess tab shows only on Apache or LiteSpeed; elsewhere set `tab` to `true` in
  `config/renatio/seomanager/htaccess.php`. Editing stays off until switched on in the tab.
- `seo:migrate-tables`, `seo:patch`, `seo:import-cms` and `seo:import-static` ask for confirmation and exit 1 when it
  is declined or cannot be asked: pass `--force` in deploy scripts, or drop `seo:migrate-tables` where `october:migrate`
  adds the columns.
- The sitemap is rebuilt on a schedule (daily at 03:00), not on save: add the `schedule:run` cron entry.
- Delete any `sitemap.xml`, `sitemap.xsl`, `llms.txt` or `llms-full.txt` in the web root; it shadows the plugin.
- Sitemap, llms.txt, IndexNow and the default robots.txt build addresses on `APP_URL`: make it the public address.
- Slug redirects and the 404 log are on; switch them off on the **General** settings tab.
- A non-GET request to a slug redirect or to a **301 Redirect** on a site host gets 308 instead of 301, so a posted form
  keeps its method and body.
- Removed: `Sitemap::buildForAllSites()` (use `rebuildAllSites()`), the `RobotsTxt` rule object (use the
  `seo_robots_txt` rule) and the **Twitter/X creator** setting with its `twitter:creator` tag.
- `$component->seoTag` is a copy of the record: compare by key, not identity. `seo.extendSeoFields` and
  `seo.extendOgFields` run on every public request, so keep them cheap.
- On a page with URL parameters a record from the page variables is used only when it holds a parameter's value
  (`/albums/:slug` and its `slug`); otherwise assign it in `seo.beforeComponentRender`.
- Sites of different groups running one theme now share hreflang links; keep brands apart with `seo.alternateSites`.
  Records of a model with the `Multisite` trait link only the versions sharing a `site_root_id`.
- The canonical tag names the record's own address and keeps `?page=N` of a paginated listing (page 1 without it);
  other parameters are dropped. Change the list in `query_params` of `config/renatio/seomanager/canonical.php`.
- A page whose canonical names another address leaves the sitemap.
- `og:locale` carries a territory (`de_DE`), error responses except 503 get `noindex, nofollow`, and the Organization
  schema carries the site URL: drop a `seo.extendJsonLd` listener that added it.
- Saving validates **Canonical URL**, **Meta Robots Advanced** (no `all` or `none`) and a 255-character limit.
- An empty body now fails **Content length**, so `seo:analyze --fail-on=poor` may start failing in CI.
- RainLab.Translate is legacy: SEO Health lists the translations only it still reads and the `translate:import-*`
  command that moves them.
