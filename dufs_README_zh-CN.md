# Dufs

[![CI](https://github.com/sigoden/dufs/actions/workflows/ci.yaml/badge.svg)](https://github.com/sigoden/dufs/actions/workflows/ci.yaml)
[![Crates](https://img.shields.io/crates/v/dufs.svg)](https://crates.io/crates/dufs)
[![Docker Pulls](https://img.shields.io/docker/pulls/sigoden/dufs)](https://hub.docker.com/r/sigoden/dufs)

Dufs 是一款独具特色的多功能文件服务器，支持静态文件托管、上传、搜索、访问控制、 WebDAV...

![demo](https://user-images.githubusercontent.com/4012553/220513063-ff0f186b-ac54-4682-9af4-47a9781dee0d.png)

## 功能特性

- 静态文件托管
- 将文件夹下载为 ZIP 文件
- 上传文件及文件夹（拖拽上传）
- 创建 / 编辑 / 搜索文件
- 支持断点续传（上传 / 下载）
- 访问控制
- 支持 HTTPS
- 支持 WebDAV
- 与 curl 配合易于使用

## 安装

### 使用 cargo

```
cargo install dufs
```

### 使用 Docker

```
docker run -v `pwd`:/data -p 5000:5000 --rm sigoden/dufs /data -A
```

### 使用 [Homebrew](https://brew.sh)

```
brew install dufs
```

### macOS / Linux / Windows 二进制

前往 [Github Releases](https://github.com/sigoden/dufs/releases) 下载对应平台版本，解压后将 dufs 加入 $PATH。

## CLI

```
Dufs is a distinctive utility file server - https://github.com/sigoden/dufs

Usage: dufs [OPTIONS] [serve-path]

Arguments:
  [serve-path]  Specific path to serve [default: .]

Options:
  -c, --config <file>        Specify configuration file
  -b, --bind <addrs>         Specify bind address or unix socket
  -p, --port <port>          Specify port to listen on [default: 5000]
      --path-prefix <path>   Specify a path prefix
      --hidden <value>       Hide paths from directory listings, e.g. tmp,*.log,*.lock
  -a, --auth <rules>         Add auth roles, e.g. user:pass@/dir1:rw,/dir2
  -A, --allow-all            Allow all operations
      --allow-upload         Allow upload files/folders
      --allow-delete         Allow delete files/folders
      --allow-search         Allow search files/folders
      --allow-symlink        Allow symlink to files/folders outside root directory
      --allow-archive        Allow download folders as archive file
      --enable-cors          Enable CORS, sets `Access-Control-Allow-Origin: *`
      --render-index         Serve index.html when requesting a directory, returns 404 if not found index.html
      --render-try-index     Serve index.html when requesting a directory, returns directory listing if not found index.html
      --render-spa           Serve SPA(Single Page Application)
      --assets <path>        Set the path to the assets directory for overriding the built-in assets
      --log-format <format>  Customize http log format
      --log-file <file>      Specify the file to save logs to, other than stdout/stderr
      --compress <level>     Set zip compress level [default: low] [possible values: none, low, medium, high]
      --completions <shell>  Print shell completion script for <shell> [possible values: bash, elvish, fish, powershell, zsh]
      --tls-cert <path>      Path to an SSL/TLS certificate to serve with HTTPS
      --tls-key <path>       Path to the SSL/TLS certificate's private key
  -h, --help                 Print help
  -V, --version              Print version
```

## 示例

以只读方式托管当前目录

```
dufs
```

允许上传 / 删除 / 搜索 / 创建 / 编辑等全部操作

```
dufs -A
```

仅允许上传

```
dufs --allow-upload
```

托管指定目录

```
dufs Downloads
```

托管单个文件

```
dufs linux-distro.iso
```

托管单页应用（React / Vue 等）

```
dufs --render-spa
```

托管静态站点（含 index.html）

```
dufs --render-index
```

设置账号密码

```
dufs -a admin:123@/:rw
```

监听指定主机和端口

```
dufs -b 127.0.0.1 -p 80
```

监听 Unix 套接字
```
dufs -b /tmp/dufs.socket
```

使用 HTTPS

```
dufs --tls-cert my.crt --tls-key my.key
```

## API

上传文件

```sh
curl -T path-to-file http://127.0.0.1:5000/new-path/path-to-file
```

下载文件
```sh
curl http://127.0.0.1:5000/path-to-file           # 下载文件
curl http://127.0.0.1:5000/path-to-file?hash      # 获取文件 sha256 哈希
```

将文件夹下载为 ZIP 文件

```sh
curl -o path-to-folder.zip http://127.0.0.1:5000/path-to-folder?zip
```

删除文件 / 文件夹

```sh
curl -X DELETE http://127.0.0.1:5000/path-to-file-or-folder
```

创建目录

```sh
curl -X MKCOL http://127.0.0.1:5000/path-to-folder
```

将文件 / 文件夹移动到新的路径

```sh
curl -X MOVE http://127.0.0.1:5000/path -H "Destination: http://127.0.0.1:5000/new-path"
```

列出 / 搜索目录内容

```sh
curl http://127.0.0.1:5000?q=Dockerfile           # 搜索文件，类似 `find -name Dockerfile`
curl http://127.0.0.1:5000?simple                 # 仅输出名称，类似 `ls -1`
curl http://127.0.0.1:5000?json                   # 以 JSON 格式输出路径
```

使用授权（支持基础或摘要认证）

```sh
curl http://127.0.0.1:5000/file --user user:pass                 # 基础认证
curl http://127.0.0.1:5000/file --user user:pass --digest        # 摘要认证
```

断点续传 - 下载

```sh
curl -C- -o file http://127.0.0.1:5000/file
```

断点续传 - 上传

```sh
upload_offset=$(curl -I -s http://127.0.0.1:5000/file | tr -d '\r' | sed -n 's/content-length: //p')
dd skip=$upload_offset if=file status=none ibs=1 | \
  curl -X PATCH -H "X-Update-Range: append" --data-binary @- http://127.0.0.1:5000/file
```

健康检查

```sh
curl http://127.0.0.1:5000/__dufs__/health
```

<details>
<summary><h2>高级主题</h2></summary>

### 访问控制

Dufs 支持基于账号的访问控制，可通过 `--auth`/`-a` 来控制谁可以对哪些路径进行哪些操作。

```
dufs -a admin:admin@/:rw -a guest:guest@/
dufs -a user:pass@/:rw,/dir1 -a @/
```

1. 使用 `@` 分隔账号和路径；省略账号代表匿名用户。
2. 使用 `:` 分隔账号的用户名和密码。
3. 使用 `,` 分隔多条路径。
4. 在路径后追加 `:rw`/`:ro` 可设置权限：`读-写` / `只读`，`:ro` 可省略。

- `-a admin:admin@/:rw`：admin 对所有路径拥有完整权限。
- `-a guest:guest@/`：guest 对所有路径仅有只读权限。
- `-a user:pass@/:rw,/dir1`：user 对 `/*` 拥有读写权限，对 `/dir1/*` 拥有只读权限。
- `-a @/`：所有路径公开，任何人可查看 / 下载。

**账号权限受 dufs 的全局权限限制。**
若 dufs 未通过 `--allow-upload` 启用上传权限，即使账号被授予 `read-write`（`:rw`）权限，也无法上传文件。

#### 密码哈希

Dufs 支持 sha-512 哈希密码。

创建哈希密码：

```sh
$ openssl passwd -6 123456 # 或 `mkpasswd -m sha-512 123456`
$6$tWMB51u6Kb2ui3wd$5gVHP92V9kZcMwQeKTjyTRgySsYJu471Jb1I6iHQ8iZ6s07GgCIO69KcPBRuwPE5tDq05xMAzye0NxVKuJdYs/
```

使用哈希密码：

```sh
dufs -a 'admin:$6$tWMB51u6Kb2ui3wd$5gVHP92V9kZcMwQeKTjyTRgySsYJu471Jb1I6iHQ8iZ6s07GgCIO69KcPBRuwPE5tDq05xMAzye0NxVKuJdYs/@/:rw'
```
> 哈希密码包含 `$6`，某些 shell 会将其展开为变量，因此需使用**单引号**包裹。

哈希密码注意事项：

1. Dufs 仅支持 sha-512 哈希密码，请确保密码字符串以 `$6$` 开头。
2. 摘要认证功能无法与哈希密码同时正常工作。

### 隐藏路径

Dufs 可通过 `--hidden <glob>,...` 选项在目录列表中隐藏指定路径。

```
dufs --hidden .git,.DS_Store,tmp
```

> `--hidden` 中的 glob 仅匹配文件和目录名称，而非完整路径，因此 `--hidden dir1/file` 无效。

```sh
dufs --hidden '.*'                          # 隐藏点文件
dufs --hidden '*/'                          # 隐藏所有目录
dufs --hidden '*.log,*.lock'                # 按扩展名隐藏
dufs --hidden '*.log' --hidden '*.lock'
```

### 日志格式

Dufs 支持通过 `--log-format` 自定义 HTTP 日志格式。

日志格式可使用以下变量：

| 变量         | 描述                                          |
| ------------ | --------------------------------------------- |
| $remote_addr | 客户端地址                                    |
| $remote_user | 认证时提供的用户名                            |
| $request     | 完整的原始请求行                              |
| $status      | 响应状态码                                    |
| $http_       | 任意请求头字段，示例：$http_user_agent, $http_referer |

默认日志格式为：`'$remote_addr "$request" $status'`。
```
2022-08-06T06:59:31+08:00 INFO - 127.0.0.1 "GET /" 200
```

关闭 http 日志
```
dufs --log-format=''
```

记录 user-agent
```
dufs --log-format '$remote_addr "$request" $status $http_user_agent'
```
```
2022-08-06T06:53:55+08:00 INFO - 127.0.0.1 "GET /" 200 Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.0.0 Safari/537.36
```

记录 remote-user
```
dufs --log-format '$remote_addr $remote_user "$request" $status' -a /@admin:admin -a /folder1@user1:pass1
```
```
2022-08-06T07:04:37+08:00 INFO - 127.0.0.1 admin "GET /" 200
```

## 环境变量

所有选项都可以通过前缀为 `DUFS_` 的环境变量设置。

```
[serve-path]                DUFS_SERVE_PATH="."
    --config <file>         DUFS_CONFIG=config.yaml
-b, --bind <addrs>          DUFS_BIND=0.0.0.0
-p, --port <port>           DUFS_PORT=5000
    --path-prefix <path>    DUFS_PATH_PREFIX=/dufs
    --hidden <value>        DUFS_HIDDEN=tmp,*.log,*.lock
-a, --auth <rules>          DUFS_AUTH="admin:admin@/:rw|@/" 
-A, --allow-all             DUFS_ALLOW_ALL=true
    --allow-upload          DUFS_ALLOW_UPLOAD=true
    --allow-delete          DUFS_ALLOW_DELETE=true
    --allow-search          DUFS_ALLOW_SEARCH=true
    --allow-symlink         DUFS_ALLOW_SYMLINK=true
    --allow-archive         DUFS_ALLOW_ARCHIVE=true
    --enable-cors           DUFS_ENABLE_CORS=true
    --render-index          DUFS_RENDER_INDEX=true
    --render-try-index      DUFS_RENDER_TRY_INDEX=true
    --render-spa            DUFS_RENDER_SPA=true
    --assets <path>         DUFS_ASSETS=./assets
    --log-format <format>   DUFS_LOG_FORMAT=""
    --log-file <file>       DUFS_LOG_FILE=./dufs.log
    --compress <compress>   DUFS_COMPRESS=low
    --tls-cert <path>       DUFS_TLS_CERT=cert.pem
    --tls-key <path>        DUFS_TLS_KEY=key.pem
```

## 配置文件

可通过 `--config <path-to-config.yaml>` 指定并使用配置文件。

配置项如下：

```yaml
serve-path: '.'
bind: 0.0.0.0
port: 5000
path-prefix: /dufs
hidden:
  - tmp
  - '*.log'
  - '*.lock'
auth:
  - admin:admin@/:rw
  - user:pass@/src:rw,/share
  - '@/'  # 根据 YAML 规范，此处必须加引号。
allow-all: false
allow-upload: true
allow-delete: true
allow-search: true
allow-symlink: true
allow-archive: true
enable-cors: true
render-index: true
render-try-index: true
render-spa: true
assets: ./assets/
log-format: '$remote_addr "$request" $status $http_user_agent'
log-file: ./dufs.log
compress: low
tls-cert: tests/data/cert.pem
tls-key: tests/data/key_pkcs1.pem
```

### 自定义 UI

Dufs 允许使用你自己的资源文件夹自定义界面。

```
dufs --assets my-assets-dir/
```

> 若只需在当前 UI 基础上微调，可以复制 dufs 的 [assets](https://github.com/sigoden/dufs/tree/main/assets) 目录并做相应修改。当前 UI 未使用任何框架，仅包含原生 HTML / JS / CSS，具备基础前端开发知识即可轻松改动。

你的资源目录中必须包含 `index.html` 文件。

`index.html` 可使用以下占位变量来获取内部数据：

- `__INDEX_DATA__`: 目录列表数据
- `__ASSETS_PREFIX__`: 资源 URL 前缀

</details>

## 著作权

Copyright (c) 2022-2024 dufs-developers.

dufs 在 MIT 许可证或 Apache License 2.0 下发布，您可任选其一。

详见 LICENSE-APACHE 与 LICENSE-MIT 文件了解授权详情。