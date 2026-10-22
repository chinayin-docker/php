# PHP 7.4 安全补丁队列

本目录存放 PHP 7.4 在 EOL 之后的官方安全补丁，由 `7.4/debian/{fpm,cli}/Dockerfile`
在 `docker-php-source extract` 之后按 `*.patch` 字典序依次 `patch -p1` 应用。

## 构建期约束

- 补丁必须能干净应用到 `php-7.4.33.tar.xz`（Dockerfile 里 `--forward --batch`，冲突即失败）
- 目录一起进构建上下文（CI 在 "Copy files" 步骤 `cp -rf 7.4/patches <workdir>/`）
- 新增或修改补丁后必须重新生成校验文件：

```bash
cd 7.4/patches && sha256sum *.patch > patches.sha256
```

## 补丁清单

| 文件 | 来源 | 说明 |
|---|---|---|
| `0001-CVE-2026-109618-109619.patch` | php-src 分支 `PHP-7.4-CVE-2026-109618-109619`，commit `712e60f9ed` | 修复 CVE-2026-109618 / CVE-2026-109619（PHP2Shell）。17 文件 / +240 −59，官方定位 "Minimal mitigation for EOL version" |
| `0002-openssl3-sslv23-padding.patch` | 照抄 php-src `PHP-8.1` 上游写法（`ext/openssl/openssl.c`） | 底包升级到 bookworm（OpenSSL 3.0）所需：`RSA_SSLV23_PADDING` 已在 OpenSSL 3.0 移除，用 `#if defined(...)` 包裹常量注册 |

上游导出命令：

```bash
curl -fsSL "https://github.com/php/php-src/commit/712e60f9edfd.patch" \
  -o 0001-CVE-2026-109618-109619.patch
```

## 版本号

版本号不靠补丁维护：Dockerfile 里 `sed` 把 `configure.ac` 的 `AC_INIT` 改成 `7.4.34`
后跑 `./buildconf --force`，configure 运行时生成 `main/php_version.h`，所以
`php -v` 输出 `PHP 7.4.34`。详见仓库根 `7.4/README-security.md`。

## 验证（构建后）

```bash
php -v | head -1                                   # PHP 7.4.34
php -i | grep max_multipart_body_parts             # 补丁指纹
php -d memory_limit=4M -r 'str_repeat("x",10485760);'; echo $?   # 期望 1
```
