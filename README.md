# WP PHPUnit Documentation

## TOC

- [Overview](#overview)
- [Installation](#installation)
- [PHPUnit Compatibility](#phpunit-compatibility)
- [Configuration](#configuration)
- [Examples](#examples)

## Overview

WP PHPUnit is the WordPress PHPUnit unit test library included in the WordPress core develop repository, made installable via Composer. The primary goal is to make getting up and running with your tests much faster. Rather than tracking down a script or copying from project to project, you have a single package to add to your development dependencies.

**Why?**

The core PHPUnit test library is a fundamental library for writing PHPUnit tests with WordPress. Historically, it has only been available to install via SVN checkout, usually from a `install-wp-tests.sh` script.
As a PHP _dependency_ it only makes sense that this should be installable via Composer with the rest of your dependencies.

One obstacle to achieving this is due to a hardcoded required placement of the `wp-tests-config.php` file by the library's `bootstrap.php`. 
When including as a library in a project, it expects the tests config file to be in the parent directory of the library's `includes` directory.

```
/ (library root)
├── data
├── includes
│   ├── bootstrap.php
│   ├── factory.php
│   ├── functions.php
│   ├── install.php
│   └── ...
└── wp-tests-config.php
```

Normally this file would not be editable installed as a Composer dependency, because files within its `vendor` directory are not intended to be modified.

WP PHPUnit solves this by including this file in the package source which acts as a kind of proxy to your "real" tests config file, which can be wherever you wish.

## Installation

Install the latest version

```sh
composer require wp-phpunit/wp-phpunit yoast/phpunit-polyfills --dev
```

Because WP PHPUnit is a simple versioned subset mirror of WordPress core, it shares the same tag as the version it was built from.

Install the version for a specific version of WordPress

```sh
composer require wp-phpunit/wp-phpunit:4.9.1 --dev
```

### `yoast/phpunit-polyfills`

As of WordPress 5.8.2 (and the latest versions of previous major releases), the [Yoast PHPUnit Polyfills](https://github.com/Yoast/PHPUnit-Polyfills) package was added as a required dependency of the PHPUnit infrastructure in WordPress core which makes up the source of WP PHPUnit. Unlike normal Composer packages, this will not be installed automatically for you as it is not a dependency of `wp-phpunit/wp-phpunit`. Just like WordPress itself it must be provided separately, although not to worry as this is very easy and can be done as part of the initial install as well (see above).

If you're already using WP PHPunit and are seeing errors about this after upgrading, simply require the package with Composer as a dev dependency and you should be all set:

```sh
composer require --dev yoast/phpunit-polyfills
```

See https://github.com/Yoast/PHPUnit-Polyfills for more information.

## PHPUnit Compatibility

The library is compatible with PHPUnit 3.6+, 4, 5, and support for newer versions of PHPUnit have been added since.

* Support for PHPUnit 6 was added in WordPress 4.8
* Support for PHPUnit 7 was added in WordPress 5.1
* Support for PHPUnit 8 and 9 is coming in WordPress 5.9
    * See [#46149](https://core.trac.wordpress.org/ticket/46149)
    * For more information about the loosened restriction for PHPUnit see https://github.com/WordPress/wordpress-develop/commit/8def694fe4c5df95f8e20e40389faf9cb92b6dca

## Configuration

The only configuration necessary for WP PHPUnit is to set the location to your real tests config file as an **environment variable**.

There are many ways to do this, but the easiest way is to define this in your `phpunit.xml` file for your project!

```xml
<phpunit>
    <!-- ... -->

    <php>
        <env name="WP_PHPUNIT__TESTS_CONFIG" value="tests/wp-config.php" />
    </php>

    <!-- ... -->
</phpunit>
```

Alternatively, you could set this in PHP anywhere before the `includes/bootstrap.php` file is required:

```php
putenv( 'WP_PHPUNIT__TESTS_CONFIG=path/to/wp-tests-config.php' );
```

Create a `wp-tests-config.php` file

```sh
curl -sSL -o wp-tests-config.php https://github.com/WordPress/wordpress-develop/raw/master/wp-tests-config-sample.php
```

Modify this file as necessary as you will likely want to commit it to version control. Be sure not to hardcode any sensitive data into it. Instead you may consider using something like [`phpdotenv`](https://github.com/vlucas/phpdotenv) to set your keys and secrets in a `.env` file which are then loaded as environment variables.

### Table Prefix (optional)

You may also configure the table prefix which you want to use for your tests.

This is configurable by setting the `WP_PHPUNIT__TABLE_PREFIX` environment variable. If set, this will take precedence over what is set in your tests config file. If not, **and the `$table_prefix` variable is not set in your tests config file** then it will use `wptests_` as a fallback. 

## Examples

Updating the default `bootstrap.php` file to use WP PHPUnit

https://github.com/wp-cli/scaffold-command/blob/04be3128e743a0d94a1db5b7da942b538e96e6a8/templates/plugin-bootstrap.mustache

```diff
--- /tmp/tests/bootstrap-before.php
+++ /tmp/tests/bootstrap-after.php
@@ -5,7 +5,9 @@
  */
 
-$_tests_dir = getenv( 'WP_TESTS_DIR' );
+// Composer autoloader must be loaded before WP_PHPUNIT__DIR will be available
+require_once dirname( dirname( __FILE__ ) ) . '/vendor/autoload.php';
+$_tests_dir = getenv( 'WP_TESTS_DIR' ) ?: getenv( 'WP_PHPUNIT__DIR' );
 
 if ( ! $_tests_dir ) {
     $_tests_dir = rtrim( sys_get_temp_dir(), '/\\' ) . '/wordpress-tests-lib';

```

**Complete Working Examples**

- [`wp-phpunit/example-project`](https://github.com/wp-phpunit/example-project)
- [`wp-phpunit/example-plugin`](https://github.com/wp-phpunit/example-plugin)

**WP PHPUnit in the Wild**

- [Packages that require wp-phpunit](https://packagist.org/packages/wp-phpunit/wp-phpunit/dependents)
