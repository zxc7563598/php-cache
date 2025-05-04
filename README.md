# hejunjie/cache

<div align="center">
  <a href="./README.md">English</a>｜<a href="./README.zh-CN.md">简体中文</a>
  <hr width="50%"/>
</div>

A layered caching system built with the decorator pattern. Supports combining memory, file, local, and remote caches to improve hit rates and simplify cache logic.

---

This is a multi-layer caching system I built myself, mainly to flexibly combine different cache layers (like memory, Redis, and file) in my projects.

The project requirements were quite diverse, and the caching tools provided by frameworks were either too limited or lacked transparency. So, I decided to build my own solution—something tailored to my own needs.
If you happen to have similar requirements, I hope this package proves useful to you 🙌.

## Features

- Supports multi-layer cache combinations with a nested cache structure, becoming more stable as it goes deeper.
- Based on the decorator pattern, extending new types of caches is very easy.
- Memory cache supports hit rate statistics, making it easier to observe and optimize.
- File cache supports concurrent locks, making it suitable for data caching in CLI scenarios.
- Custom data source interface—just implement `Interfaces\DataSourceInterface` to integrate any data source.


## Installation

Install via Composer:

```bash
composer require hejunjie/cache
```

## Quick Start

**Note**: For ease of extension, the code only implements the cache layer (memory/redis/file). In actual use cases, it is recommended to complete the data layer yourself. The example code is shown below.

```php
<?php
use Hejunjie\Cache;

// Create a cache structure: Memory -> Redis -> File -> Database
$cache = new Cache\MemoryCache(
    new Cache\RedisCache(
        new Cache\FileCache(
            new MyDataSource(), // A data source that implements the DataSourceInterface.
            '[File] Cache folder path',
            '[File] Cache duration (seconds)'
        ),
        '[Redis] Configuration'
        '[Redis] Prefix'
        '[Redis] Persistent connection'
    ),
    '[Memory] Cache duration (seconds)',
    '[Memory] Cache quantity (to prevent memory overflow)'
);

$data = $cache->get('user:123'); // Automatically performs a layer-by-layer lookup, with cache misses propagating down to the underlying data source.
```

## Custom Data Sources

As long as you implement the following interface, it can be used as the "final data source" for the cache:

For example, you can connect to a database, an API, or even another caching system.

```php
<?php

// Custom Data Sources - database
class MyDataSource implements \Hejunjie\Tools\Cache\Interfaces\DataSourceInterface
{
    protected DataSourceInterface $wrapped;
    
    // Constructor, no need for a constructor if it's the last layer.
    // public function __construct(
    //     DataSourceInterface $wrapped
    // ) {
    //     $this->wrapped = $wrapped;
    // }

    public function get(string $key): ?string
    {
        // Get the corresponding content from the database based on the key.
        // Return the content as a string string.

        // If the next layer returns data, store it in the current layer. If it's the last layer, the following code is not needed.
        // $content = $this->wrapped->get($key);
        // if ($content !== null) {
        //     $this->set($key, $content);
        // }
        // return $content;

    }

    public function set(string $key, string $value): bool
    {
        // Store the value in the database based on the key.
        // Return the storage result as `bool`.
    }

    public function del(string $key, string $value): void
    {
        // Perform a delete operation based on the key.
        // No need to return any value.
    }
}

```

## Purpose & Motivation

This package was originally built for a few of my own side projects, so I didn’t focus too much on making it universally applicable at first. But as I kept improving it, things got smoother, so I decided to organize it and share it.

If you happen to need multi-layer caching, or you're interested in the decorator pattern, feel free to give it a try.

If you have any questions or suggestions, feel free to open an issue or submit a PR — I’ll do my best to respond.

## 🔧 Additional Toolkits (Can be used independently or installed together)

This project was originally extracted from [hejunjie/tools](https://github.com/zxc7563598/php-tools).
To install all features in one go, feel free to use the all-in-one package:

```bash
composer require hejunjie/tools
```

Alternatively, feel free to install only the modules you need：

[hejunjie/utils](https://github.com/zxc7563598/php-utils) - A lightweight and practical PHP utility library that offers a collection of commonly used helper functions for files, strings, arrays, and HTTP requests—designed to streamline development and support everyday PHP projects.

[hejunjie/cache](https://github.com/zxc7563598/php-cache) - A layered caching system built with the decorator pattern. Supports combining memory, file, local, and remote caches to improve hit rates and simplify cache logic.

[hejunjie/china-division](https://github.com/zxc7563598/php-china-division) - Regularly updated dataset of China's administrative divisions with ID-card address parsing. Distributed via Composer and versioned for use in forms, validation, and address-related features

[hejunjie/error-log](https://github.com/zxc7563598/php-error-log) - An error logging component using the Chain of Responsibility pattern. Supports multiple output channels like local files, remote APIs, and console logs—ideal for flexible and scalable logging strategies.

[hejunjie/mobile-locator](https://github.com/zxc7563598/php-mobile-locator) - A mobile number lookup library based on Chinese carrier rules. Identifies carriers and regions, suitable for registration checks, user profiling, and data archiving.

[hejunjie/address-parser](https://github.com/zxc7563598/php-address-parser) - An intelligent address parser that extracts name, phone number, ID number, region, and detailed address from unstructured text—perfect for e-commerce, logistics, and CRM systems.

[hejunjie/url-signer](https://github.com/zxc7563598/php-url-signer) - A PHP library for generating URLs with encryption and signature protection—useful for secure resource access and tamper-proof links.

[hejunjie/google-authenticator](https://github.com/zxc7563598/php-google-authenticator) - A PHP library for generating and verifying Time-Based One-Time Passwords (TOTP). Compatible with Google Authenticator and similar apps, with features like secret generation, QR code creation, and OTP verification.

[hejunjie/simple-rule-engine](https://github.com/zxc7563598/php-simple-rule-engine) - A lightweight and flexible PHP rule engine supporting complex conditions and dynamic rule execution—ideal for business logic evaluation and data validation.

👀 All packages follow the principles of being lightweight and practical — designed to save you time and effort. They can be used individually or combined flexibly. Feel free to ⭐ star the project or open an issue anytime!

---

This library will continue to be updated with more practical features. Suggestions and feedback are always welcome — I’ll prioritize new functionality based on community input to help improve development efficiency together.
