---
title: 'Upgrade Guide'
intro: A guide for upgrading your existing Statamic v2.x projects to v3.0.
template: page
updated_by: 3a60f79d-8381-4def-a970-5df62f0f5d56
updated_at: 1568743749
id: f12f8ba3-19ff-48cb-a07b-653b05082d7e
blueprint: page
---
## Overview
Statamic 3 takes everything you love about v2, rewrites all the old stuff (Laravel 5.1 and Vue.js 1) with the latest hotness, adds roughly 80 fantastic new features, and speeds up performance by 5x. What's not to love about that?

> You can install our [migration tools package](https://github.com/statamic/migrator) to automate upgrading your Statamic v2 site to v3!
>
> ``` bash
> composer require statamic/migrator
> php please migrate:site
> ```

## Breaking Changes: Core

First, let's look at the breaking changes on the core application side. These would be changes that affect the control panel, front-end, cli, and generally anything that doesn't require custom PHP.

### General

- `site/content` directory is now `content`
- `site/users` directory is now `users`
- `site/settings/*.yaml` have moved to php config files in `config/statamic`. See [settings](#settings).

### Theming and Views

- The concept of "themes" is gone. Your site just has the one, and it's in `resources`.
- Templates (views) are now located in the `resources/views` instead of separate `templates`, `layouts`, and `partials` directories.
- Removed `theme:partial` tag in favor of `partial`.
- The `content` field is no longer automatically parsed for Antlers. You can flag it for parsing in the [blueprint](/blueprints). (As well as any other fields)

### Collections

- The `folder.yaml` is now up one directory so it's not mixed in amongst all the entries.
- It's also renamed to the handle of the collection. eg `blog.yaml`
- Instead of _all_ values in the YAML file cascading to entries, only values nested inside `data` will.
- Date based collections should have `date: true`, instead of `order: date`
- Ordered collections should have `orderable: true`, instead of `order: numeric`
- Mounting to a page is reversed. Instead of adding `mount: collection` to an page, you now add `mount: page-id` to the collection.
- Instead of `fieldset: foo`, you should use `blueprints: [foo]` (it's plural, and an array).

### Globals

- Data gets nested inside a `data` key. This separates the data from the meta data, which you don't really want to be used as global variables.
- When using [multiple sites](/globals#localization), the data gets stored in completely separate files.


### Users

- The locale preference for the CP has become an actual [preference](/preferences). Move `locale: en` from the top level of the user file into the `preferences` array.

### Fieldtypes

- Array fieldtype terminology was changed from `value => text` to `key => value`, and thus config options were also changed to `add_button`, `key_header` and `value_header` to match the new terminology.
- Relate and Suggest fieldtypes has been removed in favor of the [Relationship](/fieldtypes/relationship) fieldtype

### Fieldsets

Fieldsets technically still exist, although they are are a now a smaller, companion feature to Blueprints. Blueprints get attached to content. Fieldsets are an optional feature and can be used inside blueprints.

- In content etc, you should reference `blueprint: foo`  instead of `fieldset: foo`.
- There is no more `fieldsets` fieldtype. You should use the `blueprints` fieldtype.
- Field conditions use a slightly different syntax for multiple `OR` conditions and null/empty checks.
- Consider using the `statamic:migrate:fieldset` command to convert your v2 fieldsets to blueprints.

### Tags

- `get_values`
    - Removed. Use `get_content`.
- `entries`, `entries:listing`, `pages`
    - Removed. Use `collection`.
- `get_content`
    - Now only accepts IDs, not URLs.
- `relate`
    - No longer inherits paramaters from the `collection` tag.
    - You can probably remove this tag entirely, as the values should be [augmented](/augmentation).
- `form:submissions`
    - No longer inherits parameters from the `collection` tag.
- `form`
    - Replaced `name` element in `fields` array with `handle`.
    - Replaced `field` element in `fields` array with a [renderable view](/tags/form).

#### Tag Conditions

Content tag conditions still exist, but now use our new content query builders under the hood.  It's worth noting that some of these conditions may differ in behavior slightly, as they are now implemented using more agnostic query builder compatible comparisons or regular expressions instead of PHP logic.  That said, the most notable breaking changes are as follows:

- All comparisons and regex patterns are now case-insensitive by default.
- `is`, `equals`, `not`, `isnt`, and `aint` no longer work with `_strict` modifier.
- `contains` and `doesnt_contain` no longer work on array/object data.
- `matches`, `match`, `regex` and `doesnt_match` now ignore pattern delimiters and modifiers.
- `is_uppercase`, `is_lowercase`, `is_today`, `is_yesterday`, `is_between`, `is_leap_year`, `is_weekday`, `is_weekend`, `in_array` and `is_json` conditions were removed.

---


## Breaking Changes: API

### Global Functions

Most of the global functions have been removed in an effort to prevent pollution of the global namespace.

- Removed `array_get()`, use `Statamic\Support\Arr::get()`
- Removed `array_reindex()`
- Removed `array_filter_key()`
- Removed `translate()`, use `trans()`
- Removed `translate_choice()`, use `trans_choice()`
- Removed `bool_str()`, use `Statamic\Support\Str:bool()`
- Removed `str_bool()`, use `Statamic\Support\Str::toBool()`
- Removed `site_locale()`, use `Statamic\Facades\Site::current()->handle()`
- Removed `default_locale()`, use `Statamic\Facades\Config::getDefaultLocale()`
- `cp_resource_url()` is now `Statamic\Statamic::assetUrl()` (being consistent with [what Laravel considers an asset](https://laravel.com/docs/5.8/mix))
- `resource_url()` is now `Statamic\Statamic::url()`
- Removed `path()`, use `Statamic\Facades\Path::tidy($from . '/' . $to)`
- Removed `root_path()`, use `base_path()`
- Removed `webroot_path()`, use `public_path()`
- Removed `site_path()`
- Removed `local_path()`
- Removed `bundles_path()`
- Removed `addons_path()`
- Removed `settings_path()`
- Removed `site_storage_path()`
- Removed `cache_path()`
- Removed `logs_path()`
- Removed `temp_path()`
- Removed `carbon()`, use `Illuminate\Support\Carbon`
- Removed `datastore()`
- Removed `site_root()`
- Removed `resources_root()`
- Removed `collect_content()`
- Removed `collect_entries()`, use `EntryCollection::make()`
- Removed `collect_globals()`, use `GlobalCollection::make()`
- Removed `collect_assets()`, use `AssetCollection::make()`
- Removed `collect_terms()`, use `TermCollection::make()`
- Removed `collect_users()`, use `UserCollection::make()`
- Removed `collect_files()`, use `FileCollection::make()`
- Removed `collect_pages()`
- Removed `me()`, use `Statamic\Facades\User::current()`
- Removed `addon()`
- Removed `nav()`
- Removed `col_class()`
- Removed `svg()`
- Removed `inline_svg()`
- Removed `is_id()`
- Removed `active_for()`
- Removed `nav_is()`
- Removed `format_url()`, use `Statamic\Facades\URL::format()`
- Removed `markdown()`, use `Statamic\Support\Html::markdown()`
- Removed `textile()`, use `Statamic\Support\Html::textile()`
- Removed `t()`, use `__()`
- Removed `slugify()`, use `Statamic\Support\Str::slug()`
- Removed `smartypants()`, use `Statamic\Support\Html::smartypants()`
- Removed `bool()`, use `Statamic\Support\Str::toBool()`
- Removed `int()`, use `intval()`
- Removed `d()`, use `dump()`
- Removed `gravatar()`, use `Statamic\Support\URL::gravatar()`
- Removed `format_input_options()` from both PHP and JS (use `HasInputOptions` mixin)
- Removed `format_update()`
- Removed `modify()`
- Removed `refreshing_addons()`
- Removed `cp_middleware()`
- Removed `sanitize()`, use `Statamic\Support\Str::sanitize()`
- Removed `sanitize_array()`, use `Statamic\Support\Arr::sanitize()`
- Removed `array_filter_recursive`
- Removed `array_filter_use_both`, use `array_filter($arr, $cb, ARRAY_FILTER_USE_BOTH)`
- Removed `mb_str_word_count`
- Removed `to_moment_js_date_format`


### Suggest Modes

Suggest modes have been replaced by [customized relationship fieldtypes](/guide/extending/relationship-fieldtypes.html).

### Statamic\API

`Statamic\API` classes are now `Statamic\Facades`

#### Statamic\API\Addon

- Removed `manager()`
- Renamed `create` to `make`

#### Statamic\API\Arr

- Moved to `Statamic\Support\Arr`

#### Statamic\API\Asset

- Renamed `whereId` to `findById`
- Renamed `whereUrl` to `findByUrl`

#### Statamic\API\AssetContainer

- Removed `wherePath`
- Renamed `create` to `make`

#### Statamic\API\Assets

- Removed, use `Asset`.

#### Statamic\API\Auth

- Removed. Use `Illuminate\Support\Facades\Auth`

#### Statamic\API\AssetContainer

- Removed `create`, use `make`.
- Removed `wherePath`

#### Statamic\API\Cache

- Removed. Use `Illuminate\Support\Facades\Cache`
- `put()` would accept a third argument for minutes, `null` for forever. The Laravel class requires a time. Use `Cache::forever()` to put forever.

#### Statamic\API\Collection

- Renamed `create` to `make`.
- Removed `handleExists`
- Renamed `whereHandle` to `findByHandle`

#### Statamic\API\Content

- Removed. Use `Data`.

#### Statamic\API\Cookie

- Removed. Use `Illuminate\Support\Facades\Cookie`

#### Statamic\API\Crypt

- Removed. Use `Illuminate\Support\Facades\Crypt`

#### Statamic\API\Email

- Removed. Use [Laravel Mailables](https://laravel.com/docs/mail).

#### Statamic\API\Entries

- Removed. Use `Entry`

#### Statamic\API\Entry

- `whereCollection()` now only supports a single collection (use `whereInCollection()` if you need to pass multiple)
- Renamed `whereUri` to `findByUri`
- Removed `exists`
- Removed `slugExists`
- Removed `countWhereCollection()`
- Renamed `create` to `make`

#### Statamic\API\Event

- Removed. Use `Illuminate\Support\Facades\Event`

#### Statamic\API\Fieldset

- Tip: You probably want `Blueprint` now.
- Removed `create`
- Renamed `get` to `find`, and removed the `$type` argument.

#### Statamic\API\Form

- Renamed `get` to `find`
- Removed `getAllFormsets`
- Renamed `create` to `make`
- Removed `fields`

#### Statamic\API\Globals

- Removed. Use `GlobalSet`

#### Statamic\API\GlobalSet

- Renamed `create` to `make`
- Renamed `whereHandle` to `findByHandle`

#### Statamic\API\Hash

- Removed. Use `Illuminate\Support\Facades\Hash`

#### Statamic\API\Helper

- Removed

#### Statamic\API\Please

- Removed. Use `Illuminate\Support\Facades\Artisan`.

#### Statamic\API\Str

- Moved to `Statamic\Support\Str`

#### Statamic\API\URL

- Removed `removeSiteRoot()`

#### Statamic\API\Zip

- Removed because core no longer uses it. If you need it, let us know.