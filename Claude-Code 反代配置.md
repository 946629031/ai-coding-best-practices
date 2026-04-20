Claude-Code 反代配置

# CLIPROXYAPI
Docker compose 配置
```docker
services:
  cli-proxy-api:
    image: eceasy/cli-proxy-api:latest
    container_name: cliproxyapi
    ports:
      - "8317:8317"
    volumes:
      - ./config.yaml:/CLIProxyAPI/config.yaml
      - ./auth-dir:/root/.cli-proxy-api
    restart: unless-stopped
```

config.yaml
修改两处

1. Secret-key  随便设置一个密码
2. api-keys 随便设置一个 sk-xxxxxx , 后续配置 claude code 使用，进入界面后也能在配置中心添加
```docker
port: 8317
tls:
  enable: false
remote-management:
  allow-remote: true
  secret-key: 这里随便改一下
  disable-control-panel: false
  panel-github-repository: https://github.com/router-for-me/Cli-Proxy-API-Management-Center
auth-dir: /root/.cli-proxy-api
api-keys:
  - sk-xxxxxx 
debug: true
commercial-mode: false
logging-to-file: true
usage-statistics-enabled: true
force-model-prefix: false
request-retry: 3
max-retry-interval: 30
quota-exceeded:
  switch-project: true
  switch-preview-model: true
routing:
  strategy: round-robin
ws-auth: false
nonstream-keepalive-interval: 0
codex-instructions-enabled: false
```

打开 localhost:8317 , 输入你配置的密码进入，在 OAuth 登录 - Codex OAuth

打开链接，登录后把浏览器里面的回调地址填回去就可以了

# 安装 claude code
```sh
curl -fsSL https://claude.ai/install.sh | bash

# 初始化配置
# 本质上就是 在 .claude.json 配置 hasCompletedOnboarding true 来跳过 claude code 的认证
node --eval "
    const homeDir = os.homedir();
    const filePath = path.join(homeDir, '.claude.json');
    if (fs.existsSync(filePath)) {
        const content = JSON.parse(fs.readFileSync(filePath, 'utf-8'));
        fs.writeFileSync(filePath, JSON.stringify({ ...content, hasCompletedOnboarding: true }, null, 2), 'utf-8');
    } else {
        fs.writeFileSync(filePath, JSON.stringify({ hasCompletedOnboarding: true }), 'utf-8');
    }"
```

# 配置快速启动命令
ANTHROPIC_API_KEY 来自于前面配置的 cliproxy 的 config.yaml

```sh
# .zshrc or .bashrc

alias yolo="ANTHROPIC_API_KEY=sk-xxxx ANTHROPIC_BASE_URL=http://xxxxx ANTHROPIC_DEFAULT_OPUS_MODEL=gpt-5.3-codex  ANTHROPIC_DEFAULT_SONNET_MODEL=gpt-5.3-codex ANTHROPIC_DEFAULT_HAIKU_MODEL=gpt-5.4-mini claude --dangerously-skip-permissions"

source ~/.zshrc
```

- 亲测有效
  - 为啥我设置了GPT，但是打开之后还是Sonnet 4.6 呢?
  ```sh
  alias yolo="
  ANTHROPIC_API_KEY=sk-claude-proxy-local-20260416 \
  ANTHROPIC_BASE_URL=http://127.0.0.1:8317 \
  ANTHROPIC_DEFAULT_HAIKU_MODEL=gpt-5.2 \
  ANTHROPIC_DEFAULT_SONNET_MODEL=gpt-5.2 \
  claude --dangerously-skip-permissions"
  ```


# 如果一直显示未登录
```sh
rm ~/.claude.json
删除这个文件后,创建一个新的文件, 在里面写上 hasCompletedOnboarding: true, 用这种方法来跳过登录
```
