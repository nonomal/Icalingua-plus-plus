# Icalingua++ Bridge OICQ

[![GitHub package.json dependency version (prod)](https://img.shields.io/github/package-json/dependency-version/Icalingua-plus-plus/Icalingua-plus-plus/oicq?filename=icalingua-bridge-oicq%2Fpackage.json)](https://github.com/takayama-lily/oicq)

这是基于 OICQ 的 Icalingua++ 中继，可以部署在自己的服务器上，服务器通过 OICQ 库与上级服务器通信，本地电脑与自己的服务器通信。这样可以实现一些原先无法实现的功能：

-   在电脑关闭时保持在线和消息同步
-   在多个地点同时使用一个帐号

## 安装方法

### 常规安装

首先有一台服务器。

1. 服务器需要安装 `ffmpeg` 命令，选装 MongoDB / MySQL / MariaDB / PostgreSQL / Redis 作为默认数据库 SQLite 的替代。

2. 运行 `yarn` 安装必要依赖。

3. （可选）运行 `yarn compile` 将 ts 转换为 js，不转换也可运行，但是转换后效率可能高点。

4. 前往[此页面](https://paulmillr.com/ecc/)**点击按钮**生成一组 ECC 密钥对，记录私钥和 **ed** 公钥备用。

    ![ECC 密钥生成页面](https://user-images.githubusercontent.com/18461360/130779855-1cdc26ad-0037-4bbf-930c-07e8cbc2f6c9.png)

    **注意：**请不要使用自带的公钥 `207a067892821e25d770f1fba0c47c11ff4b813e54162ece9eb839e076231ab6`，它对应的私钥为 `0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef`。

    该密钥仅适用于本地环境及测试环境，暴露在公网中是非常危险的。

5. 若没有执行第二步，在 `index.ts` 所在目录创建 `config.yaml`，否则在 `build` 文件夹内创建 `config.yaml`，内容如下：

    ```yaml
    host: 0.0.0.0 # 监听地址。如果有反代工具，可以改成 localhost 或者 127.0.0.1
    pubKey: 第三步生成的公钥
    custom: false # 自定义插件功能，默认禁用
    port: 6789 # 如果需要运行多个实例，可以设置不同端口
    ```

6. 若没有执行第二步，执行 `yarn start`，否则在 `build` 文件夹中执行 `node index`，然后软件将监听在你设置的端口（默认 `6789`）。可以通过 HTTP(S) 反向代理的软件（如 caddy）将端口绑定到域名（虚拟主机）上，或者直接暴露 HTTP 端口（不建议，因为这样的话流量将以未加密的方式传输）。

#### 已知问题

如果使用 pm2 通过 `yarn start` 运行，pm2 将无法停止进程。建议使用 screen 或者编译成 js 之后再使用 pm2。

### 使用 Docker

Todo

## 客户端连接方法

首先保持 Icalingua++ 未运行状态，编辑 `~/.config/icalingua/config.yaml` 或通过 `-c` 开关自定义的配置文件，修改以下配置项：

```yaml
adapter: socketIo # 将 Icalingua++ 切换到自有协议
server: http(s)://服务器的地址，若非 HTTP80 或 HTTPS443 需要:端口号
privateKey: 安装的第三步生成的私钥
```

首次运行的时候会弹出登录界面，给 Bridge 配置账号和数据库。需要注意的是数据库是相对于服务器的地址

## 登录握手细节

客户端与服务器建立连接后，服务器将当前时间戳的 MD5 发送给客户端。客户端使用私钥签名发送给服务端验证，服务端验证成功后开放通信。
