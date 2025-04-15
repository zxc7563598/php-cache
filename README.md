# hejunjie/cache

这是我自己写的一个多层缓存系统，主要是为了在项目里能灵活组合各种缓存层（内存、Redis、文件什么的）。

项目需求比较杂，框架提供的缓存要么不够灵活，要么不够透明，于是就动手造了个轮子，也算是写给自己用的。如果你正好也有类似的需求，希望这个包能帮到你 🙌。

## 特点

- 支持多层缓存组合，套娃式缓存结构，越套越稳
- 基于装饰器模式，扩展新类型缓存很方便
- 内存缓存支持命中率统计，便于观察和调优
- 文件缓存支持并发锁，适合 CLI 场景下的数据缓存
- 自定义数据源接口，只要实现 `Interfaces\DataSourceInterface` 就能接入

## 安装

```bash
composer require hejunjie/cache
```

## 快速上手

**注意**：为了拓展方便，代码仅仅实现了缓存层（内存/redis/文件），实际应用场景中建议自行完善数据层，大概代码如下所示

```php
<?php
use Hejunjie\Cache;

// 创建一个缓存结构：Memory -> Redis -> File -> 数据库
$cache = new Cache\MemoryCache(
    new Cache\RedisCache(
        new Cache\FileCache(
            new MyDataSource(), // 实现了 DataSourceInterface 的数据源
            '[文件]缓存文件夹路径',
            '[文件]缓存时长(秒)'
        ),
        '[redis]配置'
        '[redis]前缀'
        '[redis]是否持久化链接'
    ),
    '[内存]缓存时长(秒)',
    '[内存]缓存数量(防止内存溢出)'
);

$data = $cache->get('user:123'); // 自动逐层查找，缓存 miss 会一路下沉到底层数据源
```

## 自定义数据源
只要实现下面这个接口，就可以作为缓存的“最终数据来源”使用：
比如你可以接数据库、API、甚至其他缓存系统都行。

```php
<?php

// 自定义数据源 - 数据库层
class MyDataSource implements \Hejunjie\Tools\Cache\Interfaces\DataSourceInterface
{
    protected DataSourceInterface $wrapped;
    
    // 构造函数，如果是最后一层则不需要构造函数
    // public function __construct(
    //     DataSourceInterface $wrapped
    // ) {
    //     $this->wrapped = $wrapped;
    // }

    public function get(string $key): ?string
    {
        // 根据 key 在数据库中获取对应内容
        // 返回内容字符串 `string`

        // 如果下一层返回数据，则在当前层存储。如果是最后一层则不需要下列代码
        // $content = $this->wrapped->get($key);
        // if ($content !== null) {
        //     $this->set($key, $content);
        // }
        // return $content;

    }

    public function set(string $key, string $value): bool
    {
        // 根据 key 在数据库中存储 value
        // 返回存储结果 `bool`
    }

    public function del(string $key, string $value): void
    {
        // 根据 key 在进行删除操作
        // 不需要进行返回
    }
}

```

## 用途 & 背景

这个包最初是为了我自己的几个 side project 写的，没想太多通用性，但后面越写越顺手，就整理了一下发出来。

如果你也刚好需要多层缓存，或者对装饰器模式感兴趣，可以试试看。

有啥问题或者建议都欢迎提 issue 或 PR，我会尽量回复。

## 🔧 更多工具包（可独立使用，也可统一安装）

本项目最初是从 [hejunjie/tools](https://github.com/zxc7563598/php-tools) 拆分而来，如果你想一次性安装所有功能组件，也可以使用统一包：

```bash
composer require hejunjie/tools
```

当然你也可以按需选择安装以下功能模块：

[hejunjie/china-division](https://github.com/zxc7563598/php-china-division) - 中国省市区划分数据包。

[hejunjie/error-log](https://github.com/zxc7563598/php-error-log) - 责任链日志上报系统。

[hejunjie/mobile-locator](https://github.com/zxc7563598/php-mobile-locator) - 国内手机号归属地 & 运营商识别。

[hejunjie/utils](https://github.com/zxc7563598/php-utils) - 常用工具方法集合。

[hejunjie/address-parser](https://github.com/zxc7563598/php-address-parser) - 收货地址智能解析工具，支持从非结构化文本中提取用户/地址信息。

👀 所有包都遵循「轻量实用、解放双手」的原则，能单独用，也能组合用，自由度高，欢迎 star 🌟 或提 issue。

---

该库后续将持续更新，添加更多实用功能。欢迎大家提供建议和反馈，我会根据大家的意见实现新的功能，共同提升开发效率。








