# Upgrade guide

Versions not listed here need no action. Back up the database before upgrading.

## Upgrading To 7.9.0

**A new permission is held back from every role but Developer.** `renatio.seomanager.migrate_seo_columns` governs the
**Add the missing SEO columns** button of the **Diagnostics** tab, which alters tables belonging to other plugins.
Like `change_htaccess`, it names `roles`, so October files it under Developer alone and the built-in Publisher does
not receive it — and the permissions field of a system role cannot be edited, so an editor who needs it has to be
given a custom role. Nobody loses anything they already had; the permission is new. `seo:migrate-tables` is
unaffected and stays the way to do this from a deploy script.

**A partially migrated table is now recognised and completed.** Until now a table counted as migrated as soon as it
carried `meta_title`, so a table that already had a column of its own under one of the SEO names — `meta_description`
and `og_image` are the common ones — was left with only part of the set, silently storing nothing for the rest, while
the diagnostics reported it as healthy. Both the button and `seo:migrate-tables` now compare the full set and add only
the columns that are absent. Expect the command to report work on a table it used to skip.

## Upgrading To 7.8.0

**The sitemap grows.** A CMS page whose URL carries a parameter used to be skipped whole; it is now filled with the
records of every plugin answering October's page finder (the `cms.pageLookup` events). Expect
addresses to appear that a `seo.extendSitemap` listener was written to add by hand — the listener still runs, and the
same address offered twice is written once, so nothing has to be removed. Rebuild the map after the upgrade to see it.

Only the types listing every record of their kind are read; a type addressing one record picked in the backend is
left alone. A record excluded with `noindex` cannot be recognised through the page finder, so it is listed — hide it
by keeping the plugin out of the API, or by dropping the address in a `seo.extendSitemap` listener.

**Pages nothing can list the records of are reported.** They appear on the **Sitemap** settings tab and as an
`unmapped_pages` check of the **Diagnostics** tab and of `seo:doctor`. It is a warning rather than an error, so it
does not change the exit status of the command.

## Upgrading To 7.7.1

**The `seoTags` component no longer writes on the record it renders.** The page fallback added in 7.3.0, the built
title, the robots value and the Open Graph defaults are written on a copy of the entry or the post. A layout that
prints `{{ post.meta_description }}` below the component therefore shows the value of the record again rather than the
one inherited from the page, and a `save()` on that record later in the same request no longer stores the page text in
its columns.

The copy carries the same class and the same loaded relations, so a `seo.beforeComponentRender` or `seo.extendJsonLd`
listener still sees the record it expects. It is a different instance, though — a listener that compared
`$component->seoTag` with `$this->controller->vars['post']` by identity has to compare the keys instead. The copy is
made before `seo.beforeComponentRender`, so a listener writing on `$component->seoTag` still has its value rendered,
and a record the listener puts there in its place is copied in turn.

The copy takes the attributes and the loaded relations of the record, and reads its translations from the database
like any other instance. With RainLab.Translate on a locale other than the default one, a value that some other code
wrote on the record earlier in the same request - without saving it - is therefore not carried over; set it from a
`seo.beforeComponentRender` listener instead, which runs on the copy.

**`seo.extendSeoFields` and `seo.extendOgFields` fire on the front end.** The component builds the names of the page
tags from the same events the backend forms are built from, so a listener that used to run only while a form was
rendered now runs once per rendered page. A listener doing backend work — querying the database for dropdown options,
reading `BackendAuth` — has to be made cheap or guarded, because it is on the path of every public request.

**A field added through `seo.extendSeoFields` reaches the CMS pages.** It is now offered in the SEO settings of a CMS
page, carried onto the tag the component builds from that page and inherited by an entry that leaves it blank, the
same as the fields the plugin ships. Nothing has to be configured, and a page that never held the field renders
nothing for it. A field the inspector has no control for — a `hint`, a `repeater`, a `ruler` — is left out of the page
settings, and a field added through `seo.extendOgFields` still follows the Open Graph switch in the backend forms.

## Upgrading To 7.6.0

**The social profiles are rows now.** The **Social profiles** textarea on the Organization tab is replaced by a
repeater whose rows carry the address, the service its icon is drawn from and an optional name. The migration
converts what is stored, guessing the service from the host of each address, so check the rows once after the
upgrade — an address the plugin does not recognise is given the generic globe.

Anything reading `organization_same_as` from the settings has to read `social_profiles` instead. The `sameAs`
property of the `Organization` schema is unchanged and now takes its addresses from the rows.

The field is no longer hidden by the **Structured data** switch: the new `socialProfiles` component draws the
profiles whether or not JSON-LD is enabled.

## Upgrading To 7.5.1

**`og:locale` now carries a territory.** A site whose locale names a language alone published `og:locale` as `de`,
which Open Graph ignores; it is now `de_DE`, and `og:locale:alternate` follows the same rule for the other sites of
the group. Nothing is configured for it, and `hreflang` keeps the bare language it wants. A project that needs a
different territory - `de_AT` rather than `de_DE` - overrides the map in `config/renatio/seomanager/og_locales.php`
or through the `seo.extendOgLocales` event.

**The Change .htaccess permission is no longer part of the default set of a role.** A superuser and the built-in
Developer role keep it, and any custom role that was granted it keeps it too, because a custom role stores its own
permissions. The built-in **Publisher** role loses it: October computes the permissions of its own roles on every
read and locks them in the role editor, so editors who have to reach the .htaccess editor need a custom role with
the permission ticked, or the Developer role.

**robots.txt and .htaccess end with a single newline.** Saving the settings used to write the file back without its
last byte, which showed up as a change in git on every save. The file is now left alone when nothing but that newline
differs, so the first save after the upgrade may add the newline back once.

## Upgrading To 7.5.0

**The JSON-LD tab is now the Organization tab.** The switch that turns structured data on, the warnings, the home page
preview and the organization logo all moved onto it, joined by the rest of the fields that describe who runs the site.
Nothing stored changed — only where it is edited.

The `Organization` schema and the `publisher` of an article now carry a `url` pointing at the base URL of the active
site. No configuration is involved; a project that already added the property through `seo.extendJsonLd` should drop
that listener, or it will overwrite the value the plugin sets.

Everything on the Organization tab is optional and empty on an upgrade, and an empty field is left out of the schema
rather than published blank, so the output only grows once fields are filled in. The identity stays `Organization`
until one of the LocalBusiness types is picked. Like the rest of the SEO settings, the tab is stored per site.

## Upgrading To 7.4.0

Editing `.htaccess` from the settings page is now off by default, on existing installations too. The .htaccess tab
shows a switch, **Allow editing .htaccess from this page**, in place of the editor; turning it on brings the editor
back. The switch is guarded by the existing **Change .htaccess** permission, so a user who could not edit the file
before cannot enable it either.

Nothing on disk changes and no data is lost — only the editor is hidden until you ask for it.

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
