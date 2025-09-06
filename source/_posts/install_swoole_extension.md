---
title: 手动编译安装Swoole扩展
date: 2025-4-20 12:41:53
updated: 2025-9-6 11:22:09
description: '手动编译安装Swoole扩展并启用原生CURL支持，swoole_curl_setopt(): option[10100] is not supported解决'
---
### swoole_curl_setopt(): option[10100] is not supported解决
### 安装 Swoole 扩展并启用 CURL 支持

在宝塔面板中，PHP 通常位于 `/www/server/php/你的PHP版本/bin` 目录下，例如 `/www/server/php/82/bin/phpize`。你需要根据你安装的 PHP 版本调整命令。

#### 1. **下载 Swoole 源码**

从 [Swoole GitHub](https://github.com/swoole/swoole-src/releases) 下载最新的源码包。选择 `Source code(tar.gz)` 格式的文件进行下载。

```bash
wget https://github.com/swoole/swoole-src/archive/refs/tags/v6.0.2.tar.gz
```

#### 2. **解压源码包**

假设你已经下载了 `swoole-src-6.0.2.tar.gz` 文件，首先进入 PHP 安装目录：

```bash
cd /www/server/php/82
```

然后解压下载的 Swoole 源码包：

```bash
tar -zxvf swoole-src-6.0.2.tar.gz
```

#### 3. **进入源码目录**

解压后，进入 Swoole 源码目录：

```bash
cd swoole-src-6.0.2
```

#### 4. **执行 phpize**

在编译 Swoole 扩展之前，需要先执行 `phpize` 命令来准备 PHP 扩展的编译环境。`phpize` 是 PHP 提供的一个工具，用于准备 PHP 扩展的编译过程，生成必要的配置文件。执行命令：

```bash
/www/server/php/82/bin/phpize
```

> **原理**：`phpize`是PHP扩展开发工具，作用是将Swoole源码转换为PHP可识别的扩展模块。需使用对应PHP版本的phpize，否则会出现ABI版本不匹配错误。`phpize` 会检测当前的 PHP 环境并生成一个 `configure` 脚本，使得 Swoole 能够与 PHP 正常兼容。


#### 5. **配置编译选项**

执行 `configure` 命令时，加入 `--enable-swoole-curl` 参数来启用原生的 CURL 支持。并通过 `--with-php-config` 指定 PHP 配置工具的路径。执行以下命令：

```bash
./configure --enable-swoole-curl --enable-openssl --with-php-config=/www/server/php/82/bin/php-config
  make && make install
```

> **原理**：`--enable-swoole-curl` 参数启用了 Swoole 扩展中的 CURL 支持，`--with-php-config` 则指定了 PHP 配置工具的路径，确保编译过程中 PHP 的正确配置,通过configure脚本生成Makefile，参数决定扩展功能和依赖库绑定方式。使用系统CURL可避免自研实现的兼容性问题。

#### 6. **编译和安装**

完成配置后，执行 `make` 和 `make install` 命令来编译并安装 Swoole 扩展：

```bash
make && make install
```

> **原理**：`make` 会编译 Swoole 扩展，`make install` 会将编译好的扩展安装到 PHP 扩展目录中。如果编译顺利，你会看到类似以下的输出，表示安装成功。

如果编译时提示缺少 OpenSSL 开发库，需要先安装：

CentOS/RHEL:
```bash
yum install openssl-devel
```

Ubuntu/Debian:
```bash
apt-get install libssl-dev
```

在 `/www/server/php/82/lib/php/extensions/no-debug-non-zts-20220829` 目录下，你会看到 `swoole.so` 文件：

```bash
cd /www/server/php/82/lib/php/extensions/no-debug-non-zts-20220829
```

```bash
fileinfo.so  igbinary.so  opcache.so  redis.so  swoole.so  zip.so
```

#### 7. **修改 PHP 配置文件**

在 PHP 配置文件中加载 Swoole 扩展。编辑 PHP 8.2 的 `php.ini` 文件，通常位于 `/www/server/php/82/etc/php.ini`，添加以下内容：

```bash
extension=swoole.so
```

> **原理**：这行配置指示 PHP 加载 `swoole.so` 扩展，以便 Swoole 扩展能够在 PHP 中生效。

#### 8. **重启 PHP 服务**

修改完 `php.ini` 文件后，重启 PHP 服务以应用更改：

```bash
service php-fpm restart
```

或者通过宝塔面板重启 PHP 服务。

#### 9. **验证安装**

执行以下命令来确认 Swoole 是否安装成功：

```bash
php -m | grep swoole
```
如果输出 `swoole`，说明安装成功。


验证是否成功启用 OpenSSL：
```bash
php --ri swoole | grep openssl
```

应该能看到 openssl support => enabled 的输出。

---

### 总结

通过上述步骤，你就能成功地编译并安装带有 CURL 支持的 Swoole 扩展。重要的是，确保在 `configure` 命令中添加了 `--enable-swoole-curl` 参数，这样可以启用原生 CURL 支持。编译过程中，`phpize` 和 `php-config` 工具的使用非常关键，它们帮助你确保扩展能够正确编译并与 PHP 配置兼容。

如遇到编译错误，请检查系统是否已安装 PHP 开发包和相关依赖库。