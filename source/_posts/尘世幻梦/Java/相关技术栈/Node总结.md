---
title: Node命令总结
music:
  server: netease
  type: song
  id: 373114
categories:
  - Java
  - Node
abbrlink: 141e2e2d
---

Node相关总结

<!-- more -->

# Node总结

```shell{.line-numbers}
# 查看npm版本
npm -v

# 升级到新版本
npm install npm -g

# 启动
npm run start / npm start / npm run serve

# 全局安装
npm install module_name -g

# 所在项目安装
npm install module_name

# 更新
npm update module_name -g

# 卸载
npm uninstall module_name

# 搜索
npm search module_name

# 查看全局安装模块信息，不带 -g 代表查询安装的所有模块信息
npm list -g
npm ls

# 列出全局安装的模块 带上 --depth 0 不深入到包的支点 更简洁
npm list -g --depth 0

# 查看 npm 配置
npm config list

# 检查可更新插件
npm outdated

# 设置全局配置
npm config set registry https://registry.npmmirror.com
```

> --save：将模块依赖关系写入到package.json文件的dependencies参数中
  -dev：将模块依赖关系写入到package.json文件的devDependencies参数中
  -g：表示全局
  @+version：指定版本号
> npm install 可简写为 npm i
> npm run 相关：除了 npm run start 和 npm run test，其它的命令都不能省略 run。



