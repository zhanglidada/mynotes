# 一、windows下配置claude code

## 1.1 windows + wsl 配置科学上网


1）确保 Clash Verge 允许局域网连接

打开 Clash Verge → `设置` → `网络`：

- 开启 **`允许局域网连接 (Allow LAN)`**
    
- 记下端口：HTTP 代理默认 `7890`，Socks5 默认 `7891`

2）编辑 `~/.bashrc` 文件

在 WSL 终端中运行：
```
nano ~/.bashrc
```

将以下代码块**完整复制**到文件末尾，保存（`Ctrl+O` → `回车` → `Ctrl+X`）。

```
# =====================================================
#  WSL2 代理配置 (Clash Verge @ Windows 主机)
# =====================================================

# --- 可修改的端口 ---
PROXY_HTTP_PORT=7890
PROXY_SOCKS_PORT=7891

# --- 获取 Windows 主机 IP (兼容多种方式) ---
get_windows_host_ip() {
    # 方法1：从路由表获取 (最可靠)
    local ip=$(ip route show default 2>/dev/null | awk '/default/ {print $3}')
    if [ -n "$ip" ]; then
        echo "$ip"
        return
    fi
    # 方法2：从 resolv.conf 获取
    ip=$(cat /etc/resolv.conf 2>/dev/null | grep nameserver | awk '{print $2}' | head -1)
    if [ -n "$ip" ]; then
        echo "$ip"
        return
    fi
    # 方法3：解析 mDNS 名称 (Win11 部分版本可用)
    ip=$(getent hosts host.docker.internal 2>/dev/null | awk '{print $1}')
    if [ -n "$ip" ]; then
        echo "$ip"
        return
    fi
    # 如果全部失败，返回空
    echo ""
}

# --- 开启代理 ---
proxy_on() {
    local host=$(get_windows_host_ip)
    if [ -z "$host" ]; then
        echo "[proxy] 无法获取 Windows 主机 IP，请检查 WSL 网络"
        return 1
    fi

    export http_proxy="http://${host}:${PROXY_HTTP_PORT}"
    export https_proxy="http://${host}:${PROXY_HTTP_PORT}"
    export all_proxy="socks5://${host}:${PROXY_SOCKS_PORT}"

    # 可同时设置大写变量 (部分工具只认大写)
    export HTTP_PROXY="$http_proxy"
    export HTTPS_PROXY="$https_proxy"
    export ALL_PROXY="$all_proxy"

    # 设置不需要代理的地址 (可选，按需修改)
    export no_proxy="localhost,127.0.0.1,::1,*.local,host.docker.internal"
    export NO_PROXY="$no_proxy"

    echo "[proxy] 代理已开启：http://${host}:${PROXY_HTTP_PORT}"
}

# --- 关闭代理 ---
proxy_off() {
    unset http_proxy https_proxy all_proxy
    unset HTTP_PROXY HTTPS_PROXY ALL_PROXY
    unset no_proxy NO_PROXY
    echo "[proxy] 代理已关闭"
}

# --- 显示当前代理状态 ---
proxy_status() {
    if [ -z "$http_proxy" ]; then
        echo "[proxy] 代理未开启"
    else
        echo "[proxy] 当前代理: $http_proxy"
    fi
}

# --- 默认不自动开启代理 ---
# 如果你希望每次打开终端时自动开启代理，可以取消下面这行的注释：
# proxy_on
```


3）使配置生效

运行以下命令，让修改立即生效：
```
source ~/.bashrc
```

4）测试
```
proxy_on

curl -I https://www.google.com
```


## 1.2 配置claude + deepseek



# 二、mac下配置claude code

mac 下先用brew 安装最新的node
```
brew install node
```

然后直接用npm 安装claude code
```
npm install -g @anthropic-ai/claude-code
```

接着配置claude。这里的settings json文件是一个全局配置文件，每次claude 进入项目时都能直接使用配置好的全局文件
```
mkdir -p ~/.claude
nano ~/.claude/settings.json
```

settings.json配置文件内容如下：

```
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
	/*这里是你的api key*/
    "ANTHROPIC_AUTH_TOKEN": "sk-xxx",
    // 注意，这里一定要写[1m]
    "ANTHROPIC_MODEL": "deepseek-v4-pro[1m]",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4-pro[1m]",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-v4-pro[1m]",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-flash[1m]",
    "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-v4-flash[1m]",
    "CLAUDE_CODE_EFFORT_LEVEL": "max",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
    "API_TIMEOUT_MS": "3000000"
  },
  "model": "haiku"
}

```

配置文件中的1m一定要写，主要是提醒claude code 使用1m的上下文，否则他还是会使用默认的128/256k上下文

```
```

```
```

```
```
