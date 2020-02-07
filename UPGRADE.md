# Upgrade guide

## Upgrading To 1.1.0

Plugin requires OctoberCMS build 420+ with Laravel 5.5 and PHP >=7.0.

## Upgrading To 1.2.0

Parameters for `seo.beforeComponentRender` event were changed. Now the first parameter is `$component` itself. You can access it `seoTag` property as in example in **README**. The second parameter is the `$page` variable.