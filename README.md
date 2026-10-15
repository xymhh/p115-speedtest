# p115-speedtest

测「这台机器 → 115 网盘」的**单连接上传速度**——也就是 MoviePilot 插件 CloudSubscribe
在「本地文件 → 115」这一步走的那条通道。用来横向比较不同 VPS 的上传表现。

## 一行用法

```sh
curl -sSL https://raw.githubusercontent.com/xymhh/p115-speedtest/main/p115_speedtest_vps.sh \
  | COOKIE='UID=..; CID=..; SEID=..' sh -s 100
```

想要交互式粘贴 cookie（不回显）：

```sh
curl -sSL https://raw.githubusercontent.com/xymhh/p115-speedtest/main/p115_speedtest_vps.sh -o /tmp/p115.sh
sh /tmp/p115.sh 100
```

## 它做了什么

1. 找 115 凭据：`COOKIE=` 环境变量 → 终端粘贴 → （在 MP 机器上时）扫 `/config`
2. 找可用的 p115client：本机已有就直接用；没有就用 [uv](https://docs.astral.sh/uv/)
   拉一份独立 CPython 3.12 + p115client（无 root、不动系统 python 和系统包）
3. 造 100MB 随机数据（不会命中秒传，真跑网络）上传到 115 的 `/临时测速`
4. 打印速度、判读，并输出一行 `RESULT host=.. speed=..MB/s` 方便多台机器对比
5. 自清理：本地临时文件、115 上的测试文件、uv 缓存（uv 是本次装的连 uv 一起删）

国内默认走清华 PyPI 索引 + 南大 GitHub 镜像，失败自动回退官方源。

## 环境变量

| 变量 | 默认 | 说明 |
| --- | --- | --- |
| `COOKIE` | 无 | 115 凭据 `UID=..; CID=..; SEID=..` |
| `MAX_SECONDS` | 300 | 单次上传超时上限，超了也报已传速率 |
| `KEEP_UV` | 0 | 设 1 则保留 uv 和它的缓存 |
| 第 1 个参数 | 100 | 测试大小 MB |

> 脚本内不含任何凭据，cookie 只在你本机内存里用，不落盘、不上传。
