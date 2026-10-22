# PHP 7.4 安全升级说明（7.4.33 → 7.4.34）

对应 2026-10-21 高危漏洞通报：**PHP2Shell** `CVE-2026-109618`、`CVE-2026-109619`（CVSS 9.2，已有公开 PoC 与在野利用）。

---

## 1. 背景

PHP 7.4 已于 2022-11 停止维护，官方**不会**发布 7.4.34。官方为此破例开了一个专用分支：

| 项 | 值 |
|---|---|
| 分支 | `PHP-7.4-CVE-2026-109618-109619` |
| commit | `712e60f9ed` |
| 提交说明 | `Avoid state corruption via bailout` / **"Minimal mitigation for EOL version."** |
| 规模 | 17 文件 / +240 −59 |
| 修复 | GHSA-w8hw-w9fx-j79w（CVE-2026-109618）、GHSA-6wmf-4rv5-p3rp（CVE-2026-109619） |
| tag / 二进制 | ❌ 无 —— 只能自建 |

已验证：该分支 = `php-7.4.33` 发行包 + 17 个文件的安全补丁（全树 19195/19170 个文件逐字节 sha256 比对，内容差异仅 20 个：17 补丁文件 + `NEWS`/`configure.ac`/`main/php_version.h`）。

**7.4.34 是本仓库自定的内部版本号**，用于团队按版本拉镜像，上游并不存在该 tag。

---

## 2. 本次改动

### 2.1 版本号来源

**不靠补丁维护版本号。** 决定版本的是 `configure.ac` 的 `AC_INIT`（`main/php_version.h` 是 configure 生成物，改了会被覆盖），因此：

```
sed AC_INIT: 7.4.33 -> 7.4.34  →  ./buildconf --force 重新生成 configure
```

只需改一处即得 `PHP 7.4.34`，无需维护版本补丁。

### 2.2 改动清单

| 文件 | 改动 |
|---|---|
| `7.4/patches/0001-CVE-2026-109618-109619.patch` | 新增：官方 CVE 补丁（= php-src@712e60f9ed） |
| `7.4/patches/0002-openssl3-sslv23-padding.patch` | 新增：OpenSSL 3.0 兼容（`RSA_SSLV23_PADDING` 已移除） |
| `7.4/patches/patches.sha256` | 新增：补丁完整性校验 |
| `7.4/patches/0002-openssl3-sslv23-padding.patch` | 新增：OpenSSL 3.0 兼容（`RSA_SSLV23_PADDING` 已移除） |
| `7.4/debian/{fpm,cli}/Dockerfile` | 底包 bullseye→**bookworm**；`PHP_VERSION 7.4.34`；应用补丁；`buildconf` 改版本；构建期断言；**运行时依赖保留 pipeline 对齐上游 bookworm 写法**（见 2.4） |
| `7.4/slim/{fpm,cli}/Dockerfile` | `FROM`/`PHP_VERSION`/`CYD_IMAGE_VERSION` 同步到 7.4.34-bookworm-slim |
| `.github/workflows/ci.yml` | 拷贝 `patches/` 进构建上下文；增加 `pull_request` 触发（PR 不推送镜像） |

### 2.3 底包必须升级到 bookworm（bullseye 已不可构建）

**结论：底包从 bullseye 升到 bookworm 不是可选优化，而是前置条件。**

Debian 11 (bullseye) LTS 已于 **2026-08-31 结束**，`security.debian.org` 的 bullseye 包池正在清理。实测：

| 检查 | 结果 |
|---|---|
| `deb.debian.org/debian-security/dists/bullseye-security/InRelease` | 200（索引还在） |
| `deb.debian.org/.../pool/updates/.../perl-modules-5.32_*.deb` 等实际包 | **404** |
| `archive.debian.org/debian-security/dists/bullseye-security/Release` | 200（已归档） |
| `deb.debian.org/debian-security/dists/bookworm-security/InRelease` | 200 正常 |

`apt-get install` 因此直接 `exit code 100`，镜像构建第一步就失败。

**这不是本 PR 引入的**：master 分支 2026-10-20 push 与 10-21 定时构建的全部 5 个作业（7.4/8.2/8.4 的 fpm 与 cli）均以同一原因失败：

```
X build (7.4, debian, fpm) 32s
X build (8.2, debian, fpm) 1m27s
X build (8.4, debian, cli) 1m24s
...
Err:1 http://deb.debian.org/debian-security bullseye-security/main amd64 perl-modules-5.32
    404  Not Found
```

改用 `archive.debian.org` 可以继续构建，但归档源**冻结、不再有安全更新** —— 与本次安全升级目标相悖，不可接受。

| 底包 | 工具链 | 结论 |
|---|---|---|
| bullseye (11) | gcc 10 / libxml2 2.9.10 / OpenSSL 1.1.1 | **已 EOL，无法从官方 security 源构建** |
| **bookworm (12)** | gcc 12 / libxml2 2.9.14 / OpenSSL 3.0 | **本次采用**（支持至 2028-06） |
| trixie (13) | gcc 14 / libxml2 2.14 / OpenSSL 3.5 | 不建议：GCC 14 默认把隐式函数声明当 error，PHP 7.4 代码里有 |

依赖包（`libonig-dev`、`libargon2-dev`、`libsodium-dev`、`libcurl4-openssl-dev`、`libssl-dev`、`autoconf`、`re2c`）在 bookworm 均可用。

#### bookworm 带来的两处必改项（均已落地为补丁/代码，无需手工步骤）

**（1）OpenSSL 3.0 删除 `RSA_SSLV23_PADDING`（编译期失败）**

```
ext/openssl/openssl.c:1520: 'RSA_SSLV23_PADDING' undeclared
```

该宏在 OpenSSL 1.1.1 已弃用、3.0 中删除；PHP 7.4 发布时 OpenSSL 3.0 尚未发布，故 7.4 分支未处理。
修复见 `patches/0002-openssl3-sslv23-padding.patch`，**照抄 php-src `PHP-8.1` 上游写法**用 `#if defined()` 包裹常量注册。
不自行 `#define` 假值：OpenSSL 3.0 的 EVP 层已不支持 SSLv23 padding，定义假值只会让用户拿到运行期才失败的常量。
副作用：`OPENSSL_SSLV23_PADDING` 常量在 OpenSSL 3.0 下不再注册，与 PHP 8.1+ 行为一致。

**（2）usrmerge 导致依赖清理误删运行时库（运行期失败）**

```
/usr/local/bin/php: error while loading shared libraries: libargon2.so.1
```

`apt-get purge --auto-remove` 把 `libargon2-1`/`libsodium23`/`libsqlite3-0` 等**运行时库**一并删除，镜像里 php 起不来。

根因：上游老写法拿 ldd 输出的**绝对路径**反查所属包：

```sh
# 旧写法（bookworm 上失效）
| awk '/=>/ { print $(NF-1) }' | xargs -r dpkg-query --search | cut -d: -f1
```

bookworm 默认 usrmerge，`/lib` 是 `/usr/lib` 的符号链接，dpkg 数据库记录 `/usr/lib/...` 而 ldd 输出 `/lib/...`，**绝对路径匹配不到**，包被判为无主依赖后删除。

实测对比（bookworm 容器内同一批 .so）：

| pipeline | `dpkg-query --search` 结果 |
|---|---|
| 旧：绝对路径 `/lib/aarch64-linux-gnu/libargon2.so.1` | 查询失败，只得到 `libc6` —— **漏掉 libargon2-1** |
| 新：glob `*lib/aarch64-linux-gnu/libargon2.so.1` | 正确得到 `libargon2-1:arm64`、`libc6:arm64` |

修复：该 pipeline **对齐 docker-library/php master（bookworm 分支）当前写法** —— 路径去掉 `/usr/` 与 `/` 前缀后转 glob 再查，并改用 `awk 'sub(":$","",$1)'` 解析 dpkg-query 输出。这段是上游为 bookworm/trixie 专门修过的版本，不是我们自创。

> 待跟进（超出本 PR 范围）：`8.2/8.4` 的 Dockerfile 同样在用 bullseye，目前同样构建失败，需一并迁移到 bookworm。

### 2.4 为什么不需要 bison

`$PHPIZE_DEPS` 含 `autoconf`（`buildconf` 需要），不含 bison。看起来会缺，但 PHP 的 `build/php.m4` 里：

```sh
case $php_bison_check in
  ""|invalid[)]
    if test ! -f "$srcdir/Zend/zend_language_parser.c"; then
      AC_MSG_ERROR([bison ... is required to generate PHP parsers ...])
    fi
    YACC="exit 0;"        # 解析器已预生成 → 跳过 bison
```

发行包里有预生成的解析器，所以 bison 强制检查不触发。已实测：本机 bison 仅 2.3（低于要求的 3.0），`./configure` 仍返回 0。

---

## 3. 构建期断言

Dockerfile 末尾（`php --version` 之后）：

```dockerfile
	php -v | grep -qE '^PHP 7\.4\.34 '; \
	php -i | grep -q '^max_multipart_body_parts'; \
	echo 'OK: php 7.4.34 with CVE-2026-109618/109619 fix'
```

`max_multipart_body_parts` 是官方补丁新增的 ini 项，**只有补丁真的编进产物才存在** —— 这是比版本号可靠的指纹。任一条不满足，镜像构建直接失败。

---

## 4. 构建后验证

```bash
IMG=chinayin/php:7.4.34-fpm

# 1) 版本
docker run --rm "$IMG" php -v
# 期望：PHP 7.4.34 (cli) (built: ...)        不是 7.4.31-dev / 7.4.33

# 2) 补丁指纹
docker run --rm "$IMG" php -i | grep max_multipart_body_parts
# 期望：max_multipart_body_parts => -1 => -1

# 3) OOM 行为（补丁把 OOM 改成 _exit(1)）
docker run --rm "$IMG" php -d memory_limit=4M -r 'str_repeat("x",10*1024*1024);'; echo "exit=$?"
# 期望：exit=1

# 4) 扩展集合与旧镜像一致（防止 configure 参数漏项）
docker run --rm chinayin/php:7.4.33-fpm php -m > baseline.txt
docker run --rm "$IMG" php -m | diff - baseline.txt && echo "扩展一致"
```

---

## 5. ⚠️ 三处业务可见的行为变更

官方这个补丁是"最小缓解"，代价是改变了错误处理路径。**上线前需业务确认**：

| # | 变更 | 影响 |
|---|---|---|
| 1 | **OOM 直接 `_exit(1)`** | 客户端收到 **502 而非 500**；`error_log` 里 **不再出现** `Allowed memory size exhausted`；`register_shutdown_function` 与对象析构**不再执行** → 依赖"请求结束兜底写日志/落库"的逻辑会丢数据 |
| 2 | **`session.upload_progress` 被硬禁用**（补丁里是 `if (!PS(rfc1867_enabled) \|\| true)`） | 上传进度条**静默失效**，不报错 |
| 3 | **multipart 头收紧** | 只保留 `Content-Disposition` / `Content-Type`，重复同名头被忽略；新增 `max_multipart_body_parts` 超限会截断并告警（默认 `-1` = `max_input_vars + max_file_uploads`） |

配套要求：PHP 告警口径需同步切换为 FPM 日志 `child .* exited with code 1` + 502/空响应率，否则 OOM 将进入监控盲区。

---

## 6. 安全边界说明

1. **补丁定位是"最小缓解"**，官方原文 "Minimal mitigation for EOL version"，不等价于 8.2+ 的完整修复（8.2 的完整修复 commit `3584a98bda` 涉及 389 文件 / 1029 hunk，引入 `UnwindExit` 机制，全量移植到 7.4 不可行）。
2. **扫描器从此"看不见"这个洞**：7.4.34 在上游不存在，Trivy/Grype 等会按版本号判定为不受影响。**唯一可信证据是行为指纹**（`max_multipart_body_parts` + OOM `exit=1`），这两条已进构建断言。
3. **ABI 未变**：补丁仅新增符号（`zend_mm_safe_error_ex`、`zend_new_array_singleton`），无结构体布局改动 → 现有 7.4 扩展 `.so` 可直接加载；仍建议用 §4 第 4 条比对一次扩展集。
4. **终局是升到 8.3**：7.4 已 EOL 近 4 年，本次之后还会出现下一个"只给 backport"的漏洞。本补丁是止血，不是终点。
5. **`slim` 变体编译扩展时用的是未打补丁的 7.4.33 源码**（`docker-php-source extract` 解出 tar 包原件），`PHP_VERSION_ID=70433`，ABI 无影响；如需一致可把 `/usr/src/php.tar.xz` 换成打过补丁的源码重新打包。

---

## 7. 回滚

镜像不可变，tag 切回 `7.4.33-fpm` / `7.4.33-cli` 即回滚。运行期缓解配置（`memory_limit=-1`、`session.upload_progress.enabled=0` 等）与版本无关，回滚后仍生效。

---

## 8. 后续上游更新

分支若追加 7.4 的 backport，导出新补丁追加到 `7.4/patches/`（按文件名顺序应用），更新校验文件后重建即可：

```bash
curl -fsSL "https://github.com/php/php-src/commit/<new-sha>.patch" \
  -o 7.4/patches/0002-<desc>.patch
cd 7.4/patches && sha256sum *.patch > patches.sha256
```
