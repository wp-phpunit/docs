# WP PHPUnit Documentation

## TOC

- [Overview](#overview)
- [Installation](#installation)
- [Configuration](#configuration)
- [Examples](#examples)

## Overview

WP PHPUnit is the WordPress PHPUnit unit test library included in the WordPress core develop repository, made installable via Composer. The primary goal is to make getting up and running with your tests much faster. Rather than tracking down a script or copying from project to project, you have a single package to add to your development dependencies.

This project is not intended to be a drop-in replacement for the ubiquitous `install-wp-tests.sh` script but aimed at simplifying and streamlining local development.

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
composer require wp-phpunit/wp-phpunit --dev
```

Because WP PHPUnit is a simple versioned subset mirror of WordPress core, it shares the same tag as the version it was built from.

Install a the version for a specific version of WordPress

```sh
composer require wp-phpunit/wp-phpunit:4.9.1 --dev
```

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

Modify this file as necessary as you will likely want to commit it to version control. Be sure not to hardcode any sensitive data into it. Instead you may consider using something like [`phpdotenv`](https://github.com/vlucas/phpdotenv) to set your keys and secrets in a `.env` file which is then loaded as environment values.

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

**Example Project**

Check out the [`wp-phpunit/example-project`](https://github.com/wp-phpunit/example-project) for a complete working example.