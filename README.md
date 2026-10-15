# p115-speedtest

测「这台机器 → 115 网盘」的**单连接上传速度**——也就是 MoviePilot 插件 CloudSubscribe
在「本地文件 → 115」这一步走的那条通道。用来横向比较不同 VPS 的上传表现。

## 一行用法

```sh
# 国外 / 能直连 GitHub 的机器
curl -sSL https://raw.githubusercontent.com/xymhh/p115-speedtest/main/p115_speedtest_vps.sh \
  | COOKIE='UID=..; CID=..; SEID=..' sh -s 100

# 国内机器：换成任意一个代理前缀（四个都已实测可取到本脚本）
curl -sSL https://cdn.jsdelivr.net/gh/xymhh/p115-speedtest@main/p115_speedtest_vps.sh -o /tmp/p115.sh
sh /tmp/p115.sh 100
```

国内镜像前缀（拼在 `https://raw.githubusercontent.com/xymhh/p115-speedtest/main/p115_speedtest_vps.sh` 前面）：

| 前缀 | 备注 |
| --- | --- |
| `https://ghfast.top/` | 首选，实测已取到最新版 |
| `https://gh-proxy.com/` | 首选，实测已取到最新版 |
| `https://ghproxy.net/` | 备用（有缓存，可能滞后几小时） |
| （jsDelivr） | `https://cdn.jsdelivr.net/gh/xymhh/p115-speedtest@main/p115_speedtest_vps.sh`，不走 GitHub，但缓存可达 12h |

多镜像自动重试（推荐，脚本下载失败会自动换下一个）：

```sh
for u in \
  https://ghfast.top/https://raw.githubusercontent.com/xymhh/p115-speedtest/main/p115_speedtest_vps.sh \
  https://gh-proxy.com/https://raw.githubusercontent.com/xymhh/p115-speedtest/main/p115_speedtest_vps.sh \
  https://ghproxy.net/https://raw.githubusercontent.com/xymhh/p115-speedtest/main/p115_speedtest_vps.sh \
  https://cdn.jsdelivr.net/gh/xymhh/p115-speedtest@main/p115_speedtest_vps.sh \
  https://gh-proxy.com/https://raw.githubusercontent.com/xymhh/p115-speedtest/main/p115_speedtest_vps.sh ; do
  curl -fsSL --max-time 30 "$u" -o /tmp/p115.sh && [ -s /tmp/p115.sh ] && break
done
sh /tmp/p115.sh 100
```

## 它做了什么

1. 找 115 凭据：`COOKIE=` 环境变量 → 终端粘贴 → （在 MP 机器上时）扫 `/config`
2. 找可用的 p115client：本机已有就直接用；没有就用 [uv](https://docs.astral.sh/uv/)
   拉一份独立 CPython 3.12 + p115client（无 root、不动系统 python 和系统包）
3. 造 100MB 随机数据（不会命中秒传，真跑网络）上传到 115 的 `/临时测速`
4. 打印速度、判读，并输出一行 `RESULT host=.. speed=..MB/s` 方便多台机器对比
5. 自清理：本地临时文件、115 上的测试文件、uv 缓存与它装的 python（uv 是本次装的连 uv 一起删）

国内环境已内置回退：pip 走清华索引、python 3.12 走南大 python-build-standalone 镜像、
uv 二进制走 ghproxy，任一步失败自动换官方源。

## 环境变量

| 变量 | 默认 | 说明 |
| --- | --- | --- |
| `COOKIE` | 无 | 115 凭据 `UID=..; CID=..; SEID=..` |
| `MAX_SECONDS` | 300 | 单次上传超时上限，超了也报已传速率 |
| `KEEP_UV` | 0 | 设 1 则保留 uv 和它的缓存 |
| 第 1 个参数 | 100 | 测试大小 MB |

> 脚本内不含任何凭据，cookie 只在你本机内存里用，不落盘、不上传。
