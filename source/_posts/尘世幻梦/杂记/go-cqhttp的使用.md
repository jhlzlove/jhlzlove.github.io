---
title: go-cqhttp的使用
categories: qq机器人
---

到 [go-cqhttp](https://github.com/Mrs4s/go-cqhttp/releases/tag/v1.0.0-rc1) 下载最新版本。一般的 Linux 系统选择 **go-cqhttp_linux_amd64.tar.gz** 包下载即可。

```bash
cd ~ && mkdir go-cqhttp && cd go-cqhttp

# 下载go-cqhttp最新的rc1版本
wget https://github.com/Mrs4s/go-cqhttp/releases/download/v1.0.0-rc1/go-cqhttp_linux_amd64.tar.gz

# 解压，给予权限
tar xf go-cqhttp_linux_amd64.tar.gz
chmod +x go-cqhttp
```

编辑 `config.yml`，如果自己没有创建，在第一次 `./go-cqhttp` 启动时会自动生成，然后修改即可。关于 [配置文件](https://docs.go-cqhttp.org/guide/config.html) 可以在官方文档中了解详情。

```yml{.line-numbers}
account: # 账号相关
  uin: 123456789 # QQ账号
  password: '123456789' # 密码为空时使用扫码登录
  encrypt: false  # 是否开启密码加密
  status: 0      # 在线状态
  relogin:
    delay: 3
    interval: 3
    max-times: 0

  use-sso-address: true

heartbeat:
  interval: 5

message:
  post-format: string
  ignore-invalid-cqcode: false
  force-fragment: true
  fix-url: false
  proxy-rewrite: ''
  report-self-message: false
  remove-reply-at: false
  extra-reply-data: false
  skip-mime-scan: false

output:
  log-level: warn
  log-aging: 15
  log-force-new: true
  log-colorful: true
  debug: false

default-middlewares: &default
  access-token: ''
  filter: ''
  rate-limit:
    enabled: false
    frequency: 1
    bucket: 1

database:
  leveldb:
    enable: true
  cache:
    image: data/image.db
    video: data/video.db

servers:
  - ws-reverse:
      universal: ws://127.0.0.1:6660/ws/
      api: ws://127.0.0.1:6660/api/
      event: ws://127.0.0.1:6660/event/
      reconnect-interval: 3000
      middlewares:
        <<: *default
```

首次运行时，先验证qq账号。 运行 `./go-cqhttp`，完成验证码或二维码登录验证，验证完成后，Ctrl+c退出，然后再 `nohup ./go-cqhttp &` 让 gocq 进入后台运行。

至此，go-cqhttp 的环境就运行好了，有关依赖于 go-cqhttp 的插件就可以运行起来，实现相应的功能了。
