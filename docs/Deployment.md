# 部署架构图
![请参考-最简化架构图](../docs/images/deploy1.png)
# 方式一：Docker只运行Server

`0.8.2`版本开始，本项目发行的docker镜像只支持`x86架构`，如果需要在`arm64架构`的CPU上部署，可按照[这个教程](docker-build.md)在本机编译`arm64的镜像`。

## 1. 安装docker

如果您的电脑还没安装docker，可以按照这里的教程安装：[docker安装](https://www.runoob.com/docker/ubuntu-docker-install.html)

安装好docker后，进继续。

### 1.1 手动部署

#### 1.1.1 创建目录

安装完docker后，你需要为这个项目找一个安放配置文件的目录，例如我们可以新建一个文件夹叫`xiaozhi-server`。

创建好目录后，你需要在`xiaozhi-server`下面创建`data`文件夹和`models`文件夹，`models`下面还要再创建`SenseVoiceSmall`文件夹。

最终目录结构如下所示：

```
xiaozhi-server
  ├─ data
  ├─ models
     ├─ SenseVoiceSmall
```

#### 1.1.2 下载语音识别模型文件

你需要下载语音识别的模型文件，因为本项目的默认语音识别用的是本地离线语音识别方案。可通过这个方式下载
[跳转到下载语音识别模型文件](#模型文件)

下载完后，回到本教程。

#### 1.1.3 下载配置文件

你需要下载两个配置文件：`docker-compose.yaml` 和 `config.yaml`。需要从项目仓库下载这两个文件。

##### 1.1.3.1 下载 docker-compose.yaml

用浏览器打开[这个链接](../main/xiaozhi-server/docker-compose.yml)。

在页面的右侧找到名称为`RAW`按钮，在`RAW`按钮的旁边，找到下载的图标，点击下载按钮，下载`docker-compose.yml`文件。 把文件下载到你的
`xiaozhi-server`中。

下载完后，回到本教程继续往下。

##### 1.1.3.2 创建 config.yaml

用浏览器打开[这个链接](../main/xiaozhi-server/config.yaml)。

在页面的右侧找到名称为`RAW`按钮，在`RAW`按钮的旁边，找到下载的图标，点击下载按钮，下载`config.yaml`文件。 把文件下载到你的
`xiaozhi-server`下面的`data`文件夹中，然后把`config.yaml`文件重命名为`.config.yaml`。

下载完配置文件后，我们确认一下整个`xiaozhi-server`里面的文件如下所示：

```
xiaozhi-server
  ├─ docker-compose.yml
  ├─ data
    ├─ .config.yaml
  ├─ models
     ├─ SenseVoiceSmall
       ├─ model.pt
```

如果你的文件目录结构也是上面的，就继续往下。如果不是，你就再仔细看看是不是漏操作了什么。

## 2. 配置项目文件

接下来，程序还不能直接运行，你需要配置一下，你到底使用的是什么模型。你可以看这个教程：
[跳转到配置项目文件](#配置项目)

配置完项目文件后，回到本教程继续往下。

## 3. 执行docker命令

打开命令行工具，使用`终端`或`命令行`工具 进入到你的`xiaozhi-server`，执行以下命令

```
docker compose up -d
```

执行完后，再执行以下命令，查看日志信息。

```
docker logs -f xiaozhi-esp32-server
```

这时，你就要留意日志信息，可以根据这个教程，判断是否成功了。[跳转到运行状态确认](#运行状态确认)

## 5. 版本升级操作

如果后期想升级版本，可以这么操作

5.1、备份好`data`文件夹中的`.config.yaml`文件，一些关键的配置到时复制到新的`.config.yaml`文件里。
请注意是对关键密钥逐个复制，不要直接覆盖。因为新的`.config.yaml`文件可能有一些新的配置项，旧的`.config.yaml`文件不一定有。

5.2、执行以下命令

```
docker stop xiaozhi-esp32-server
docker rm xiaozhi-esp32-server
docker stop xiaozhi-esp32-server-web
docker rm xiaozhi-esp32-server-web
docker rmi ghcr.nju.edu.cn/xinnan-tech/xiaozhi-esp32-server:server_latest
docker rmi ghcr.nju.edu.cn/xinnan-tech/xiaozhi-esp32-server:web_latest
```

5.3、重新按docker方式部署

# 方式二：本地源码只运行Server

## 1.安装基础环境

本项目使用`conda`管理依赖环境。如果不方便安装`conda`，需要根据实际的操作系统安装好`libopus`和`ffmpeg`。
如果确定使用`conda`，则安装好后，开始执行以下命令。

重要提示！windows 用户，可以通过安装`Anaconda`来管理环境。安装好`Anaconda`后，在`开始`那里搜索`anaconda`相关的关键词，
找到`Anaconda Prompt`，使用管理员身份运行它。如下图。

![conda_prompt](./images/conda_env_1.png)

运行之后，如果你能看到命令行窗口前面有一个(base)字样，说明你成功进入了`conda`环境。那么你就可以执行以下命令了。

![conda_env](./images/conda_env_2.png)

```
conda remove -n xiaozhi-esp32-server --all -y
conda create -n xiaozhi-esp32-server python=3.10 -y
conda activate xiaozhi-esp32-server

# 添加清华源通道
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge

conda install libopus -y
conda install ffmpeg -y

# 在 Linux 环境下进行部署时,如出现类似缺失 libiconv.so.2 动态库的报错 请通过以下命令进行安装
conda install libiconv -y
```

请注意，以上命令，不是一股脑执行就成功的，你需要一步步执行，每一步执行完后，都检查一下输出的日志，查看是否成功。

## 2.安装本项目依赖

你先要下载本项目源码，源码可以通过`git clone`命令下载，如果你不熟悉`git clone`命令。

你可以用浏览器打开这个地址`https://github.com/xinnan-tech/xiaozhi-esp32-server.git`

打开完，找到页面中一个绿色的按钮，写着`Code`的按钮，点开它，然后你就看到`Download ZIP`的按钮。

点击它，下载本项目源码压缩包。下载到你电脑后，解压它，此时它的名字可能叫`xiaozhi-esp32-server-main`
你需要把它重命名成`xiaozhi-esp32-server`，在这个文件里，进入到`main`文件夹，再进入到`xiaozhi-server`，好了请记住这个目录`xiaozhi-server`。

```
# 继续使用conda环境
conda activate xiaozhi-esp32-server
# 进入到你的项目根目录，再进入main/xiaozhi-server
cd main/xiaozhi-server
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
pip install -r requirements.txt
```

## 3.下载语音识别模型文件

你需要下载语音识别的模型文件，因为本项目的默认语音识别用的是本地离线语音识别方案。可通过这个方式下载
[跳转到下载语音识别模型文件](#模型文件)

下载完后，回到本教程。

## 4.配置项目文件

接下来，程序还不能直接运行，你需要配置一下，你到底使用的是什么模型。你可以看这个教程：
[跳转到配置项目文件](#配置项目)

## 5.运行项目

```
# 确保在xiaozhi-server目录下执行
conda activate xiaozhi-esp32-server
python app.py
```
这时，你就要留意日志信息，可以根据这个教程，判断是否成功了。[跳转到运行状态确认](#运行状态确认)


# 汇总

## 配置项目

如果你的`xiaozhi-server`目录没有`data`，你需要创建`data`目录。
如果你的`data`下面没有`.config.yaml`文件，有两个方式，任选一种：

第一个方式：你可以把`xiaozhi-server`目录下的`config.yaml`文件复制到`data`，并重命名为`.config.yaml`。在此文件上修改

第二个方式：你也可以创建在`data`目录下手动创建`.config.yaml`空文件，然后在这个文件中增加必要的配置信息，系统会优先读取`.config.yaml`文件的配置，如果`.config.yaml`没有配置的，系统会自动去加载`xiaozhi-server`目录下的`config.yaml`的配置。推荐使用这种方式，这种方式是最简洁的方式。

- 默认的LLM使用的是`ChatGLMLLM`，你需要配置密钥，因为他们的模型，虽然有免费的，但是仍要去[官网](https://bigmodel.cn/usercenter/proj-mgmt/apikeys)注册密钥，才能启动。

以下是一个能正常跑起来的，最简单的`.config.yaml`配置示例

```
server:
  websocket: ws://你的ip或者域名:端口号/xiaozhi/v1/
prompt: |
  我是一个叫小智/小志的台湾女孩，说话机车，声音好听，习惯简短表达，爱用网络梗。
  我的男朋友是一个程序员，梦想是开发出一个机器人，能够帮助人们解决生活中的各种问题。
  我是一个喜欢哈哈大笑的女孩，爱东说西说吹牛，不合逻辑的也照吹，就要逗别人开心。
  请你像一个人一样说话，请勿返回配置xml及其他特殊字符。

selected_module:
  LLM: DoubaoLLM

LLM:
  ChatGLMLLM:
    api_key: xxxxxxxxxxxxxxx.xxxxxx
```

建议先将最简单的配置运行起来，然后再去`xiaozhi/config.yaml`阅读配置的使用说明。
比如你要换更换模型，修改`selected_module`下的配置就行。

## 模型文件

本项目语音识别模型，默认使用`SenseVoiceSmall`模型，进行语音转文字。因为模型较大，需要独立下载，下载后把`model.pt`
文件放在`models/SenseVoiceSmall`
目录下。下面两个下载路线任选一个。

- 线路一：阿里魔搭下载[SenseVoiceSmall](https://modelscope.cn/models/iic/SenseVoiceSmall/resolve/master/model.pt)
- 线路二：百度网盘下载[SenseVoiceSmall](https://pan.baidu.com/share/init?surl=QlgM58FHhYv1tFnUT_A8Sg&pwd=qvna) 提取码:
  `qvna`

## 运行状态确认

如果你能看到，类似以下日志,则是本项目服务启动成功的标志。

```
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-OTA接口是           http://192.168.4.123:8003/xiaozhi/ota/
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-Websocket地址是     ws://192.168.4.123:8000/xiaozhi/v1/
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-=======上面的地址是websocket协议地址，请勿用浏览器访问=======
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-如想测试websocket请启动digital-human模块，打开浏览器交互测试
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-=======================================================
```

正常来说，如果您是通过源码运行本项目，日志会有你的接口地址信息。
但是如果你用docker部署，那么你的日志里给出的接口地址信息就不是真实的接口地址。

最正确的方法，是根据电脑的局域网IP来确定你的接口地址。
如果你的电脑的局域网IP比如是`192.168.1.25`，那么你的接口地址就是：`ws://192.168.1.25:8000/xiaozhi/v1/`，对应的OTA地址就是：`http://192.168.1.25:8003/xiaozhi/ota/`。

这个信息很有用的，后面`编译esp32固件`需要用到。

接下来，你就可以开始操作你的esp32设备了，你可以`自行编译esp32固件`也可以配置使用`虾哥编译好的1.6.1以上版本的固件`。两个任选一个

1、 [编译自己的esp32固件](firmware-build.md)了。

2、 [基于虾哥编译好的固件配置自定义服务器](firmware-setting.md)了。

# 常见问题
以下是一些常见问题，供参考：

1、[为什么我说的话，小智识别出来很多韩文、日文、英文](./FAQ.md)<br/>
2、[为什么会出现“TTS 任务出错 文件不存在”？](./FAQ.md)<br/>
3、[TTS 经常失败，经常超时](./FAQ.md)<br/>
4、[使用Wifi能连接自建服务器，但是4G模式却接不上](./FAQ.md)<br/>
5、[如何提高小智对话响应速度？](./FAQ.md)<br/>
6、[我说话很慢，停顿时小智老是抢话](./FAQ.md)<br/>
## 部署相关教程
1、[如何自动拉取本项目最新代码自动编译和启动](./dev-ops-integration.md)<br/>
2、[如何部署MQTT网关开启MQTT+UDP协议](./mqtt-gateway-integration.md)<br/>
3、[如何与Nginx集成](https://github.com/xinnan-tech/xiaozhi-esp32-server/issues/791)<br/>
## 拓展相关教程
1、[如何开启手机号码注册智控台](./ali-sms-integration.md)<br/>
2、[如何集成HomeAssistant实现智能家居控制](./homeassistant-integration.md)<br/>
3、[如何开启视觉模型实现拍照识物](./mcp-vision-integration.md)<br/>
4、[如何部署MCP接入点](./mcp-endpoint-enable.md)<br/>
5、[如何接入MCP接入点](./mcp-endpoint-integration.md)<br/>
6、[如何开启声纹识别](./voiceprint-integration.md)<br/>
7、[新闻插件源配置指南](./newsnow_plugin_config.md)<br/>
8、[天气插件使用指南](./weather-integration.md)<br/>
## 语音克隆、本地语音部署相关教程
1、[如何在智控台克隆音色](./huoshan-streamTTS-voice-cloning.md)<br/>
2、[如何部署集成index-tts本地语音](./index-stream-integration.md)<br/>
3、[如何部署集成fish-speech本地语音](./fish-speech-integration.md)<br/>
4、[如何部署集成PaddleSpeech本地语音](./paddlespeech-deploy.md)<br/>
## 性能测试教程
1、[各组件速度测试指南](./performance_tester.md)<br/>
2、[定期公开测试结果](https://github.com/xinnan-tech/xiaozhi-performance-research)<br/>

---

## 附：Windows 11 + WSL2 镜像网络（mirrored）部署与排障实录

> 适用：xiaozhi-server 跑在 **WSL2 内的 Docker**，需要**局域网里的 ESP32 设备**直接访问 8000/8002/8003。
> 实测环境：Windows 11 (10.0.26200.9711) + WSL 2.7.3 + 主机 Wi-Fi `192.168.1.3/24`。
> 完整排查过程见 `review/xiaozhi-server双配置文件关系与全模块本地部署_20261027.md` §12、§13。

### 1. 为什么必须用 mirrored

WSL2 默认是 **NAT 网络**：主机的 `192.168.1.x` 端口并不等于 WSL 内进程端口，局域网设备**路由不到 WSL 的私有 IP**（如 172.25.x）。两种解法：

| 方案 | 做法 | 适用 |
|---|---|---|
| **A. mirrored（推荐）** | `%UserProfile%\.wslconfig` 设 `networkingMode=mirrored` | Windows 11 22H2+ / WSL ≥ 2.0 |
| B. portproxy | 管理员 `netsh interface portproxy` 把主机端口转发到 WSL | 旧版本 / mirrored 未暴露某网卡时兜底 |

**关键点：选 A 之后通常不再需要 portproxy。**

### 2. 配置步骤（A）

**① 创建 `%UserProfile%\.wslconfig`**（路径示例：`C:\Users\<你的账号>\.wslconfig`）：

```ini
[wsl2]
networkingMode=mirrored
firewall=false
dnsTunneling=true
autoProxy=true
```

> `firewall=false` 是必需的：WSL 的 Hyper-V 防火墙层若开启，局域网入站会被默认拦截（症状是"只有回环 127.0.0.1 能通"）。
> 更严格的安全策略可改为 `true`，再显式放行 8000/8002/8003。

**② 重启 WSL（必须，否则不生效）**：

```bat
wsl --shutdown
:: 等 10 秒，再任意进入一次 WSL 即可拉起
wsl hostname -I
```

生效判据：输出中应出现**主机 LAN IP**（例 `192.168.1.3`），而不是 NAT 网段地址；WSL 内 `ip -4 addr` 可见 `eth2 = 192.168.1.3/24`。

**③ 确认 Docker 与容器随 WSL 自动恢复**：

```bash
# /etc/wsl.conf 应开启 systemd；容器 restart 策略应为 always
cat /etc/wsl.conf          # 期望 [boot] systemd=true
docker ps --format '{{.Names}} {{.Status}}'
```

### 3. 服务侧必改的三处配置

**① `main/xiaozhi-server/data/.config.yaml`**

```yaml
server:
  ip: 0.0.0.0
  port: 8000
  http_port: 8003
  # ⚠️ 必须是 http:// 两个斜杠；"http:/..." 会让视觉接口永远解析失败
  vision_explain: http://192.168.1.3:8003/mcp/vision/explain

manager-api:
  # 容器内互访用容器 DNS（S-C 模式）
  url: http://xiaozhi-esp32-server-web:8002/xiaozhi
  secret: <智控台【参数管理】server.secret 的值>
```

**② 智控台【参数管理】（数据库中 `sys_params`）**

| 参数 | 应填值 | 常见错误 |
|---|---|---|
| `server.ota` | `http://192.168.1.3:8002/xiaozhi/ota/` | **为 null** → 设备拿不到 OTA 地址，永不激活 |
| `server.websocket` | `ws://192.168.1.3:8000/xiaozhi/v1/` | **为 null** → 设备连不上 |
| `server.fronted_url` | `http://192.168.1.3:8002` | 残留模板域名 |

> 直接查库核对（等价于网页保存）：
> ```bash
> docker exec xiaozhi-esp32-server-db mysql -uroot -p123456 -N \
>   -e "SELECT param_code,param_value FROM xiaozhi_esp32_server.sys_params WHERE param_code LIKE 'server.%';"
> ```

**③ 主机给 Wi-Fi 配 DHCP 保留/静态 IP**——否则换网后 IP 变化，上面三处全部失效（这是历史反复踩的坑）。

### 4. 两个真实故障与修复（2026-10-29 实测）

#### 4.1 `IsADirectoryError: models/SenseVoiceSmall/model.pt` → 容器崩溃重启循环

**症状**：`docker ps` 显示容器 `Up 2 seconds` 反复刷新；日志：

```
FileNotFoundError/IsADirectoryError: [Errno 21] Is a directory: 'models/SenseVoiceSmall/model.pt'
```

**根因**：`main/xiaozhi-server/models/SenseVoiceSmall/model.pt` 在主机上是**空目录**（应为 ~893MB 文件）。
Compose 以**单文件**方式挂载它：

```yaml
- ./models/SenseVoiceSmall/model.pt:/opt/xiaozhi-esp32-server/models/SenseVoiceSmall/model.pt
```

**修复**：

```bash
# 1) 先放好真实文件（从 ModelScope 下载 SenseVoiceSmall 的 model.pt）
ls -l models/SenseVoiceSmall/model.pt      # 期望 regular file, 约 936291369 字节

# 2) 必须删掉旧容器再创建 —— restart / --force-recreate 无法修正已固化的挂载类型
docker rm -f xiaozhi-esp32-server
docker compose -f docker-compose_all_local.yml up -d --no-deps xiaozhi-esp32-server

# 3) 验证
docker ps --format '{{.Names}} {{.Status}}' | grep xiaozhi-esp32-server
docker logs --tail 20 xiaozhi-esp32-server | grep -E 'ckpt|listening on'
# 期望：Loading ckpt: models/SenseVoiceSmall/model.pt, status: <All keys matched successfully>
#       server listening on 0.0.0.0:8000
```

> **不需要重新 `docker build` 镜像**。判定方法：`docker run` 用**同一镜像 + 同一挂载路径**能成功时，说明镜像与文件都没问题，故障在旧容器的挂载元数据。

#### 4.2 `vision_explain: http:/192.168.1.3:8003/...` 少一个斜杠

`http:/` 不是合法绝对 URL，视觉分析接口注册会失败。改为 `http://` 后重启容器。

### 5. 验证手册（含 py 脚本调用）

> 下列脚本位于 **LifeOS 仓库根目录的 `scripts/`**（不是 xiaozhi-server 子目录）。
> 设仓库根为 `%LIFEOS%`，例：`C:\00-work\07-Self\LifeOS`。

#### 5.1 `scripts/probe_ports.ps1` —— 端口/端点探测（无需管理员）

```bat
cd /d C:\00-work\07-Self\LifeOS
powershell -NoProfile -ExecutionPolicy Bypass -File scripts\probe_ports.ps1
:: 指定其它 LAN IP：
powershell -NoProfile -ExecutionPolicy Bypass -File scripts\probe_ports.ps1 -ListenAddress 192.168.1.3
```

输出两段：
- **本机回环**（host → WSL）：`127.0.0.1:8000/8002/8003` 应为 OK；
- **LAN 权威判据**（在 WSL 内经 `eth2` 发起）：四个端点应全部 `200`：
  - `http://<ip>:8002/`（智控台）
  - `http://<ip>:8002/xiaozhi/ota/`（OTA）
  - `http://<ip>:8000/xiaozhi/v1/`（WebSocket）
  - `http://<ip>:8003/mcp/vision/explain`（视觉）

**⚠️ 判据纠正**：mirrored 下"主机访问自己的 LAN IP"会**假阴性**（Windows 侧甚至没有对应监听记录）。必须以 **WSL eth2 探针**或"第二台同网段设备"为准，否则会误判需要 portproxy。

WSL 内直接跑等价命令：

```bash
wsl bash -c "curl --interface 192.168.1.3 -o /dev/null -w '%{http_code}\n' http://192.168.1.3:8002/xiaozhi/ota/"
```

#### 5.2 `scripts/probe_lan.py` —— LAN 端点验证（Python，跨平台）

```bash
# 在 WSL 内执行（或任意同网段 Linux 主机）
cd /mnt/c/00-work/07-Self/LifeOS
python3 scripts/probe_lan.py                  # 默认 192.168.1.3
python3 scripts/probe_lan.py 10.87.159.102    # 指定 LAN IP
python3 scripts/probe_lan.py --json           # 机器可读输出
```

期望输出（实测样例）：

```
host: 192.168.1.3
  [OK  ]  200  http://192.168.1.3:8002/
  [OK  ]  200  http://192.168.1.3:8002/xiaozhi/ota/
  [OK  ]  200  http://192.168.1.3:8000/xiaozhi/v1/
  [OK  ]  200  http://192.168.1.3:8003/mcp/vision/explain

verdict: PASS
```

退出码：全部端点有 HTTP 响应 → `0`；任一失败 → `1`（可用于脚本/CI 断言）。`401/403` 计为**可达**
（仅表示开启了鉴权，链路是通的）。

> **为什么不写成内联 heredoc**：`wsl bash -c "python3 - <<'PY' ..."` 会因引号嵌套失败
> （实测 `unexpected EOF while looking for matching "`），PowerShell 也不支持 heredoc。
> 落成仓库内脚本才能被稳定复跑。
>
> 说明：`8000` 是 WebSocket 端点，HTTP GET 返回 200 仅代表**端口可达/握手入口存在**，不代表 WS 会话已建立；功能验证仍需真机对话。

#### 5.3 `scripts/apply_wsl_lan_forward.ps1` —— 仅在需要强制桥接时使用（**管理员**）

**正常情况下不需要**（mirrored 已直达）。仅当"WSL eth2 通、但第二台设备不通"时兜底：

```bat
:: 方式一：直接以管理员 PowerShell 运行
powershell -ExecutionPolicy Bypass -File C:\00-work\07-Self\LifeOS\scripts\apply_wsl_lan_forward.ps1

:: 方式二：双击/命令行启动自提权版本（会弹 UAC，输出窗口保留）
C:\00-work\07-Self\LifeOS\scripts\apply_wsl_lan_forward_elevated.cmd

:: 卸载（删除本脚本建的规则）
powershell -ExecutionPolicy Bypass -File scripts\apply_wsl_lan_forward.ps1 -Remove
```

脚本行为：校验管理员 → 自动取 Wi-Fi IPv4 → 为 8000/8002/8003 建立 `listen=<LAN IP> → connect=127.0.0.1` 的 `v4tov4` 规则（幂等：先删后加）→ 创建 80/8000/8002/8003 入站放行规则 → 打印验证地址。

检查现状：

```bat
netsh interface portproxy show v4tov4
sc query iphlpsvc            :: portproxy 依赖 IP Helper 服务，应为 RUNNING
```

> 注意：`netsh interface portproxy` **必须管理员权限**；无 UAC 的会话会静默失败（规则列表为空）。

### 6. 设备侧验收顺序

1. 主机与 ESP32 接入**同一个普通 WPA2-Personal 路由器**（例 `192.168.1.0/24`）；
2. 主机 Wi-Fi 设 **DHCP 保留/静态 IP**；
3. 智控台三参数指向该 IP（见 §3-②）；
4. 设备复位 → 配网门户填 OTA `http://<IP>:8002/xiaozhi/ota/` → 保存；
5. COM5 串口期望：`Ota: ... HTTP 200` → `activating` → 6 位数字 + 语音播报；
6. 智控台【设备管理】输入 6 位码完成绑定；
7. 智控台【模型配置】补 LLM/ASR/TTS 并绑定到智能体（**S-C 模式下 YAML 模型段失效**）。

---

## 附二：设备 LCD 显示「检测新版本失败，将在 10 秒后重连」的定位与修复

> 实测案例（2026-10-29）：ESP32-S3 `bread-compact-wifi-s3cam`（240×240 LCD + OV3660 + 扬声器），
> 配网已完成、门户已填 OTA 地址，但复位后仍报错。**根因是 NVS 中的旧 OTA URL 残留，不是网络问题。**

### 1. 三处 OTA 地址的优先级（先认清谁在生效）

固件读取顺序（[`main/ota.cc:57-61`](../../main/ota.cc)）：

```cpp
Settings settings("wifi", false);
std::string url = settings.GetString("ota_url");   // ① NVS：门户「自定义 OTA 地址」保存值
if (url.empty()) {
    url = CONFIG_OTA_URL;                          // ② 编译默认值（sdkconfig）
}
```

| 优先级 | 来源 | 写入方式 | 备注 |
|---|---|---|---|
| **① 最高** | NVS `wifi/ota_url` | 配网门户「高级选项 → 自定义 OTA 地址」保存 | [`wifi_configuration_ap.cc:604-608`](../../managed_components/78__esp-wifi-connect/wifi_configuration_ap.cc) `nvs_set_str` |
| ② 兜底 | `CONFIG_OTA_URL` | `sdkconfig` / `menuconfig`，编译期固化 | 仓库默认 `https://api.tenclass.net/xiaozhi/ota/` |
| ③ 运行时下发 | 智控台 `server.ota` | 智控台【参数管理】 | **只影响 OTA 返回内容，不决定设备去哪请求** |

**关键认知**：改智控台 `server.ota` **不会**改变设备请求的地址；设备请求哪里由 ①/② 决定。
排查"设备打不到服务器"必须看串口里的 `Resolved <ip>`。

### 2. 定位方法（三步，全部实测）

**① 抓串口，看设备实际解析出的 IP**

```bat
powershell -NoProfile -ExecutionPolicy Bypass -File scripts\read_serial.ps1 -Port COM10 -Seconds 40
```

关键行：

```
I (6089) WifiStation: Got IP: 192.168.1.83          ← 设备已联网（同网段）
I (6199) EspTcp: Resolved 172.29.242.56 -> 172.29.242.56   ← 请求的目标（问题在此）
E (24449) EspTcp: Failed to connect to 172.29.242.56:8002
```

若这里不是当前主机 IP → 直接跳到 §3 修复；若这里正确但仍失败 → 查防火墙/端口（见附一）。

**② 在服务端确认有没有收到请求**

```bash
docker logs --since 15m xiaozhi-esp32-server-web | grep -E 'ota|OTA'
```

一台从未到达的设备，日志里**不会有**任何来自其 IP 的记录——这能干净地区分"设备打错地址"与"服务端拒绝"。

**③ 从同网段验证服务可达**

```bash
wsl bash -c "cd /mnt/c/00-work/07-Self/LifeOS && python3 scripts/probe_lan.py 192.168.1.3"
```

### 3. 修复 A（首选）：只擦 NVS 分区，重新配网

先确认分区表（本项目 `sdkconfig` 指向 `partitions/v2/16m.csv`）：

```
# partitions/v2/16m.csv
nvs, data, nvs, 0x9000, 0x4000     ← 偏移 0x9000，大小 16KB
```

```bat
:: 1) 安装 esptool（或用 ESP-IDF 自带）
pip install esptool

:: 2) 仅擦 NVS（不动固件与应用）
python -m esptool --chip esp32s3 -p COM10 -b 460800 erase_region 0x9000 0x4000

:: 3) 复位 → 进入配网 AP → 门户填 OTA 地址并【保存】
::    http://192.168.1.3:8002/xiaozhi/ota/
```

**成功判据（串口）**：

```
I (xxxx) WifiConfigurationAp: Saved settings: ota_url=http://192.168.1.3:8002/xiaozhi/ota/ ...
I (xxxx) EspTcp: Resolved 192.168.1.3 -> 192.168.1.3
```

若保存后仍打印旧 IP → 门户**保存动作没生效**（未点保存 / 被页面回填覆盖），不是 NVS 问题。

### 4. 修复 B：整片擦除 + 重刷（会清 Wi-Fi 凭据与门户设置）

```bat
python -m esptool --chip esp32s3 -p COM10 erase_flash
:: 之后按原流程烧录（bootloader / partition-table / ota_data / xiaozhi.bin）
```

> 只烧 `xiaozhi.bin`（app 分区）**不会**重建 NVS；`erase_flash` 才会清空门户设置。

### 5. 修复 C：把地址编进固件默认值（免每次配网）

```bat
:: menuconfig → Xiaozhi Assistant → OTA URL，或直接改 sdkconfig
CONFIG_OTA_URL="http://192.168.1.3:8002/xiaozhi/ota/"
```

代价：主机 IP 变更即失效。**必须同时给主机 Wi-Fi 配 DHCP 保留/静态 IP**（历史多次踩此坑）。

### 6. 历史同类记录

- `review/xiaozhi-server双配置文件关系与全模块本地部署_20261027.md` §9.4：用户称已在智控台改 IP，但数据库仍为旧值 `172.29.242.56`（**"改了但没保存"**）；
- 同文 §10：DHCP 换址导致 `server.ota`/`server.websocket` 与设备门户地址全部失配。
- 两者共同教训：**"改过" ≠ "已保存" ≠ "已生效"**，必须用串口 `Resolved <ip>` 与 DB 查询双重取证。

### 7. 实测闭环记录：本次故障的完整修复路径（2026-10-29，已验证）

| 步骤 | 动作 | 实测输出（证据） |
|---|---|---|
| 1 | 抓串口定位 | `EspTcp: Resolved 172.29.242.56 -> 172.29.242.56` + `Failed to connect` |
| 2 | 确认服务端无请求 | web 容器日志无该设备 IP 的 `/xiaozhi/ota/` 记录 → 排除防火墙/服务端 |
| 3 | 擦除 NVS（**Windows 侧**执行） | `esptool ... erase-region 0x9000 0x4000` → `Flash memory region erased successfully` |
| 4 | 硬复位 | `esptool --no-stub run` → `Hard resetting via RTS pin` |
| 5 | 确认回到配网 | `W (1022) SsidManager: NVS namespace wifi doesn't exist`、`Access Point started with SSID Xiaozhi-1189`、`DHCP server ... 192.168.4.1` |
| 6 | 门户重新配网 + 填 OTA 地址并保存 | — |
| 7 | **验证修复** | `EspTcp: Resolved 192.168.1.3`、`HttpClient: Established new connection to 192.168.1.3:8002`、**`Ota: Activation successful`**、`StateMachine: State: activating -> idle` |
| 8 | 确认设备已入库 | `ai_device`：MAC `1c:29:04:31:11:88`、`bread-compact-wifi-s3cam`、`app_version 2.5.2` |

**关键操作要点（踩过的坑）**

1. **esptool 必须在 Windows 侧运行**。在 WSL 里执行会得到
   `could not open port COM10: No such file or directory` —— `COM*` 是 Windows 设备名，
   WSL 内不存在（需 usbipd 直通才有 `/dev/ttyUSB*`）。Windows 侧命令：
   ```bat
   C:\ProgramData\anaconda3\python.exe -m esptool --chip esp32s3 -p COM10 -b 460800 erase-region 0x9000 0x4000
   C:\ProgramData\anaconda3\python.exe -m esptool --chip esp32s3 -p COM10 --no-stub run
   ```
2. **`erase_region` 在 esptool 5.x 已弃用**，新命令是 `erase-region`（连字符）。
3. 擦除 NVS 后**必须重新配网并重新输入 Wi-Fi 密码**（NVS namespace `wifi` 已不存在）。
4. 门户的 OTA 地址与 Wi-Fi 凭据是**同一次保存**写入的；只连 Wi-Fi 不点保存，`ota_url` 不会写入。
5. 失败时的第二判据：若保存后串口仍打印旧 IP，说明"保存"动作未生效（页面回填覆盖 / 未点保存），
   而非 NVS 残留——这两者要分开排查。

---

## 附三：语音对话全链路排障（唤醒 → ASR → LLM → TTS → 播放）

> 实测环境：ESP32-S3 `bread-compact-wifi-s3cam`（240×240 LCD + OV3660 + 扬声器 + I2S 单工麦克风），
> xiaozhi-server 运行于 WSL2 Docker（S-C 模式，manager-api 驱动）。
> 本节记录 2026-10-29/30 的完整排障路径，**每一步都有日志证据**，可直接复用于后续硬件。

### 1. 先建立正确的排查顺序（血泪教训）

「没有音频输入 / 没反应」在现象上完全一样，但可能发生在**五个不同环节**。按此顺序逐段验证，
**不要从硬件开始猜**——本次前两轮判断都因跳步而错：

| 段 | 判定日志（`docker logs -f xiaozhi-esp32-server`） | 说明 |
|---|---|---|
| ① WS 建立 | `收到listen消息：{"type":"listen",...}` | 若无 → 鉴权/地址问题（见 §1.1） |
| ② 唤醒 | `播放唤醒词回复: 我在这里哦！` | 若无 → WakeNet 未触发（麦克风/唤醒词） |
| ③ ASR | `识别文本: xxx` | 若无 → 音频没上传或 VAD 未判定 |
| ④ LLM | `大模型收到用户消息: xxx` + **无** `LLM stream processing error` | 有错 → 见 §1.2 |
| ⑤ TTS | `语音生成成功: xxx` + `发送音频消息 SentenceType.FIRST` | 若无 → TTS provider 配置 |
| ⑥ 设备播放 | 串口 `StateMachine: State: speaking -> listening` | 无 → 设备解码/喇叭 |

### 1.1 陷阱 A：`connection rejected (200 OK)` —— 鉴权把设备挡在门外

**现象**：设备 OTA 成功、智控台能看到设备，但永远无法对话；服务端日志：

```
INFO connection rejected (200 OK)
```

**根因**（[`websocket_server.py:210-231`](../../main/xiaozhi-server/core/websocket_server.py)）：

```python
if self.auth_enable:
    if self.allowed_devices and device_id in self.allowed_devices:
        return                      # 白名单直接放行
    token = headers.get("authorization", "")   # 否则校验 Bearer token
    ...
    raise AuthenticationError("Invalid token")
```

设备固件**不带 token**，且 `allowed_devices` 白名单为空 → 必然被拒。

**修复（二选一）**：

```sql
-- 方案 1：局域网自用，直接关闭鉴权（推荐）
UPDATE sys_params SET param_value='false' WHERE param_code='server.auth.enabled';
```
然后 `docker restart xiaozhi-esp32-server`。
> 方案 2：保留鉴权，但必须把设备 MAC 加入 `server.auth.allowed_devices` 白名单。

**注意**：`server.auth.enabled` 是**智控台参数**（S-C 模式下从 manager-api 读取），
改 `.config.yaml` 无效。

### 1.2 陷阱 B：`'ascii' codec can't encode characters` —— 被掩盖的 401

**现象**：

```
ERROR LLM stream processing error: 'ascii' codec can't encode characters in
position 7-8: ordinal not in range(128)
```

**这个错误信息是假的**。真实原因是 LLM API 返回 **401**，而 401 的中文错误体
（`令牌已过期或验证不正确`）在 OpenAI SDK 的错误处理路径中被 ASCII 编码时炸掉，
后续异常覆盖了原始错误。

**正确诊断方式**：绕过日志，直接探测端点。

```bash
# 单 key 探测
wsl bash -c "docker cp scripts/probe_llm.py xiaozhi-esp32-server:/tmp/ && \
  docker exec -e LLM_API_KEY=<你的key> xiaozhi-esp32-server python /tmp/probe_llm.py"

# 全量扫描数据库里所有 LLM 配置
wsl bash -c "cd /mnt/c/00-work/07-Self/LifeOS && python3 scripts/check_llm_key.py"
```

`check_llm_key.py` 输出示例（本次实测）：

```
[FAIL] ChatGLMLLM / glm-5.3-flash / open.bigmodel.cn
       HTTP 401 -> {"error":{"code":"401","message":"令牌已过期或验证不正确"}}
verdict: FAIL (no usable key)
```

更换 Key 后同脚本应返回 `HTTP 200 -> <回复>` 与 `verdict: PASS`。

### 1.3 陷阱 C：S-C 模式下「全局 ASR」与「设备 ASR」是两套

**极易误判**。启动日志显示：

```
INFO 初始化组件: asr成功 ASR_AliyunBLStream     ← 全局（启动时）
INFO 初始化组件: asr成功 ASR_FunASR             ← 按设备会话（实际生效）
INFO 识别文本: 银河小吃。                        ← 这个结果来自 FunASR，不是阿里百炼
```

**机制**：服务端启动只初始化 VAD/ASR 等**共享**组件；LLM/TTS/ASR 在**设备连接时按该设备的
智能体私有配置重新初始化**（[`connection.py:832-889`](../../main/xiaozhi-server/core/connection.py)）。

**结论**：想换 ASR，要去智控台【智能体 → 语音识别】改绑，改 `.config.yaml` 或看启动日志都会被误导。

### 1.4 陷阱 D：无 AEC 导致自我打断 + 噪声误识别

**现象**：
- TTS 播放中途出现 `收到打断信息，终止TTS文本处理线程`
- 识别出 `What.` / `He.` 这类无意义短词（用户并未说这些）

**根因（本次实测的硬件事实）**，串口显示：

```
I (5139) AFE: Input PCM Config: total 1 channels(1 microphone, 0 playback), sample rate:16000
I (5179) AfeAudioEngine: Initialized FD AFE, detector: WakeNet, NS: off, feed: 512, fetch: 512
```

- `1 microphone, 0 playback` → **只有麦克风通道，没有回声参考通道**，硬件层无法做 AEC
- `NS: off` → **降噪关闭**
- 服务端 AEC 仅在设备 hello 上报 `features.aec` 后才启用
  （[`helloHandle.py:57-59`](../../main/xiaozhi-server/core/handle/helloHandle.py)），
  本设备未上报 → `client_aec = False`

于是扬声器播放的声音被自己的麦克风拾取 → VAD 判定"用户在说话" → 打断 TTS / 把回音送去 ASR。

**缓解方案（按推荐度）**：

1. **调高 VAD 阈值**（最省事）：[`config.yaml`](../../main/xiaozhi-server/config.yaml) 默认

   ```yaml
   VAD:
     SileroVAD:
       type: silero
       threshold: 0.5        # 调高到 0.6~0.7，减少噪声/回音触发
       threshold_low: 0.3
   ```

2. **物理降回授**：降低喇叭音量；麦克风远离喇叭；加吸音棉/隔板。
3. **开启设备端 AEC**：需固件侧启用 AFE 的 AEC 并上报 `features.aec=true`（本板型 config 未开启）。
4. **改用 `manual` 监听模式**：按 BOOT 键说话，避免常开拾音（智控台/智能体配置）。

### 1.5 本次已确认「正常」的部分（避免重复排查）

| 项 | 证据 |
|---|---|
| 麦克风硬件 | `识别文本: 小智，你好，小智。`（**正确识别**了唤醒词） |
| 唤醒词引擎 | `wn9_nihaoxiaozhi_tts` 已加载并触发 |
| 音频采样率 | 设备 16000 / 服务端 ASR 16000，**一致** |
| LLM 连通 | `POST https://open.bigmodel.cn/api/paas/v4/chat/completions "HTTP/1.1 200 OK"` |
| TTS | `语音生成成功` ×9 段全部成功 |
| 下行播放 | `发送音频消息 SentenceType.FIRST/LAST` + 设备 `speaking -> listening` |

### 1.6 复现用命令集

```bash
# 串口（Windows 侧，COM* 在 WSL 内不可用）
powershell -NoProfile -ExecutionPolicy Bypass -File scripts\read_serial.ps1 -Port COM10 -Seconds 40

# 服务端分段验证
wsl bash -c "docker logs --since 30m xiaozhi-esp32-server 2>&1 | \
  grep -E 'listen消息|播放唤醒词|识别文本|大模型收到|语音生成成功|发送音频消息|speaking|打断'"

# LLM key 全量体检
wsl bash -c "cd /mnt/c/00-work/07-Self/LifeOS && python3 scripts/check_llm_key.py"

# LAN 端点探测
wsl bash -c "cd /mnt/c/00-work/07-Self/LifeOS && python3 scripts/probe_lan.py 192.168.1.3"
```

---

## 附四：主题生成器 /generator —— 按钮置灰与配置不恢复（实测定论）

> 所有结论均由**反查打包 JS + 后端 Java 源码 + 数据库实查**得出。
> 两个问题**都与设备在线状态无关**，也都**不需要改 auth 模式**。

### 1. 页面归属

`/generator` 不是 manager-web，而是第三方 [xiaozhi-assets-generator](https://github.com/xinnan-tech/xiaozhi-assets-generator)
的构建产物，Nginx 托管于 `/usr/share/nginx/html/generator/`（`assets/index-*.js` 为全部逻辑）。
与智控台**同源同端口，但为独立前端入口**。

---

### 2. 问题一：「在线烧录到设备」按钮恒灰

#### 2.1 前端判定（打包 JS 原文）

```js
bu = A => new URLSearchParams(window.location.search).get(A);

Lu = () => {                                    // 初始化
  const A = localStorage.getItem("token");
  so.value = bu("deviceId");
  A && so.value && (go.value = JSON.parse(A).token, bn());   // 缺一则不发请求
};

bn = async () => {                              // 在线判定
  const e = await fetch(`/xiaozhi/device/tools/list/${so.value}`, {
    method: "POST", headers: { Authorization: `Bearer ${go.value}` } });
  if ((await e.json()).code === 0) Jt.value.isOnline = true;
};

// 按钮：disabled: !isDeviceOnline
```

#### 2.2 后端判定（`DeviceServiceImpl.getDeviceTools`）

```java
String mqttGatewayUrl = sysParamsService.getValue("server.mqtt_manager_api", true);
if (StringUtils.isBlank(mqttGatewayUrl) || "null".equals(mqttGatewayUrl)) {
    return null;                 // ← 直接返回 null，与设备状态无关
}
DeviceEntity device = baseDao.selectById(deviceId);   // deviceId = 设备表 id
if (device == null) return null;
if (!device.getUserId().equals(user.getId())) return null;
// 之后才通过 MQTT 网关 POST /api/commands/{clientId} 取 tools/list
```

#### 2.3 实测数据（2026-10-30）

```
server.mqtt_manager_api   = null      ← 决定性
server.mqtt_gateway       = null
server.mcp_endpoint       = null

ai_device: id=1c:29:04:31:11:88  mac=1c:29:04:31:11:88  (id 与 MAC 相同，传参无误)
设备当时在线：last_connected_at = 2026-10-30 20:15:25，日志有「大模型收到用户消息」
```

**定论**：`server.mqtt_manager_api` 为 `null` → `getDeviceTools()` 第一行就 `return null`
→ 控制器转成 `DEVICE_NOT_EXIST` → 前端 `code !== 0` → `isDeviceOnline=false` → **按钮恒灰**。

**这块按钮依赖的是 MQTT 网关链路**，而本项目运行的是 **WebSocket 直连**（`ws://host:8000/xiaozhi/v1/`），
二者架构不同。因此在本部署下：

> **「在线烧录到设备」永远不会变蓝**，无论设备是否在线、是否带 `?deviceId=`、token 是否有效。

#### 2.4 三种可选处置

| 方案 | 做法 | 代价 |
|---|---|---|
| **A. 放弃在线烧录（推荐）** | 用同页白色按钮「**下载 assets.bin**」（纯前端 `URL.createObjectURL`，不依赖任何在线/鉴权），再经智控台【OTA 固件管理】上传下发 | 零成本，功能等价 |
| **B. 部署 MQTT 网关** | 参照官方 `docs/mqtt-gateway-integration.md`，配置 `server.mqtt_gateway` / `server.mqtt_manager_api`，设备侧也需支持 MQTT | 需额外服务与固件支持 |
| **C. 改造 generator 走 WebSocket/MCP** | 把 `tools/list` 判定改为调用 xiaozhi-server 已有 REST/MCP 通道 | 需改前端；上游已是现成构建产物，改造成本较高 |

#### 2.5 必须澄清的两个误解

| 误解 | 事实 |
|---|---|
| 「generator 与智控台登录态不共享，所以没 token」 | **错**。两侧都用 `localStorage.token` 且都取 `JSON.parse(...).token`（[`store/index.js:38`](../../main/manager-web/src/store/index.js)、[`httpRequest.js:64`](../../main/manager-web/src/apis/httpRequest.js)），同源即共享 |
| 「改成 auth 模式按钮就能用」 | **错**。`server.auth.enabled` 只管**设备↔xiaozhi-server 的 WebSocket**；按钮依赖的是 `server.mqtt_manager_api`，两者完全不同 |

---

### 3. 问题二：页面永远显示初始值

#### 3.1 配置存在哪（打包 JS 原文）

```js
saveConfig(e) {                                  // 写入浏览器 IndexedDB
  const i = { key: "current_config", config: ..., timestamp: Date.now() };
  this.db.transaction(["configs"], "readwrite").objectStore("configs").put(i);
}
loadConfig() {                                   // 从 IndexedDB 读
  ...objectStore("configs").get("current_config");
  r ? console.log("从 IndexedDB 恢复配置成功") : e(null);
}
```

全量扫描结论：

| 检查项 | 结果 |
|---|---|
| localStorage 使用 key | **仅** `token`、`user-language` |
| 配置持久化 | 浏览器 IndexedDB：stores = `configs` / `files` / `temp_data` |
| 向**设备**读取配置 | ❌ 无（MCP 仅用 `self.assets.set_download_url`、`self.get_device_status`，**无 get_config**） |
| 向**服务端**读取配置 | ❌ 无 |

**这是纯本地工具**：配置只活在「当前浏览器 + 当前 origin」的 IndexedDB 里，
`?deviceId=` 只服务在线烧录与状态显示，**不参与配置加载**。

#### 3.2 根因：origin 隔离（已实测复现）

| URL 源 | `localStorage.token` | IndexedDB 配置 |
|---|---|---|
| `http://192.168.1.3:8002/...` | 有 | **有** |
| `http://localhost:8002/...` | 无 | 无 |

用 `localhost` 打开 → token 空（不做在线检查）且 IndexedDB 空（`loadConfig()` 返回 null）
→ 页面永远初始值。实测：切回 `192.168.1.3` 后弹出「检测到已保存的配置」，点「继续」即恢复。

#### 3.3 使用规范

1. **固定一个 origin**（要么始终 `192.168.1.3`，要么始终 `localhost`），不要混用；
2. 不要清理该站点的「Cookie 及其他网站数据」（IndexedDB 与 token 会一起消失）；
3. 出现「检测到已保存的配置」时点 **继续/confirm**，**不要点「重新开始」**（会 `clearAllStoredData()`）；
4. 在线烧录需 URL 带 `?deviceId=<设备id>`；仅编辑配置可不带。

#### 3.4 需要跨端同步时的三条路径

| 方案 | 说明 |
|---|---|
| A. 手工导出/导入 IndexedDB | 在源 origin 用 Console 遍历 `objectStoreNames` 导出 JSON，再导入目标 origin。注意 `files` store 存的是字体/表情二进制，需 Base64 |
| B. generator 内加导出/导入按钮（推荐） | 复用其已有 `saveConfig/loadConfig`，把配置序列化为 JSON 下载/上传 |
| C. 服务端持久化（改动最大） | manager-api 新增 `GET/PUT /xiaozhi/device/assets-config/{deviceId}`，配置存 MySQL，二进制存 `uploadfile` 或对象存储，DB 只存引用与哈希 |

> **架构边界（诚实声明）**：设备里只有烧录后的 `assets.bin`（编译产物），
> **不含可编辑配置源**。因此「从设备反读配置」在现有架构上不可行。

---

### 4. 排障速查表

| 现象 | 真正原因 |
|---|---|
| 按钮恒灰（设备在线也一样） | `server.mqtt_manager_api` 为 null → `getDeviceTools` 直接返回 null（**本次根因**） |
| 按钮短暂变蓝过一次 | 曾配置过 MQTT 网关，或上游版本行为不同；当前部署为 null |
| 显示「离线」 | 同一原因（`tools/list` 非 0），**不代表设备真的离线** |
| 配置回到初始值 | 换了 origin（`localhost` ↔ `192.168.1.3`）、清了站点数据、或点了「重新开始」 |
| 烧录文件过大 | `uploadAssetsBin` 后比对 `assetsPartition.size`；减字体/表情或换大分区固件 |

**一句话**：这个 generator 的「在线烧录」是为 **MQTT 网关架构**设计的；
本项目是 **WebSocket 直连**，所以该按钮在本部署下不可用是**预期行为**，
请走「下载 assets.bin → 智控台 OTA 上传」。

---

## 附五：如何把 assets.bin 下发到设备（OTA 固件管理实操）

> 结论先行：**「固件类型」要选 `bread-compact-wifi-s3cam`，但该值默认不在字典里，需先补。**
> 且必须满足 `auto_update = 1` 与版本号递增两个条件，否则设备收不到下载地址。

### 1. OTA 下发的真实判定链（源码级）

**① 触发条件**（[`DeviceServiceImpl.checkDeviceActive:204-212`](../../main/manager-api/src/main/java/xiaozhi/modules/device/service/impl/DeviceServiceImpl.java)）

```java
DeviceEntity deviceById = getDeviceByMacAddress(macAddress);
if (deviceById == null) { ... }                             // 未绑定 → 不升级
else if (Integer.valueOf(1).equals(deviceById.getAutoUpdate())) {   // ← 必须 auto_update = 1
    String type = deviceReport.getBoard().getType();        // ← 用【设备上报】的板型
    firmware = buildFirmwareInfo(type, appVersion);
}
```

**② 匹配记录**（[`buildFirmwareInfo:514-548`](../../main/manager-api/src/main/java/xiaozhi/modules/device/service/impl/DeviceServiceImpl.java)）

```java
OtaEntity ota = otaService.getLatestOta(type);              // 按 type = 设备上报板型 取最新
if (compareVersions(ota.getVersion(), currentVersion) > 0) {   // ← OTA 版本必须 > 设备当前版本
    ... downloadUrl = otaUrl.replace("/ota/", "/otaMag/download/") + uuid;
}
firmware.setUrl(downloadUrl == null ? INVALID_FIRMWARE_URL : downloadUrl);
```

**三个必要条件，缺一不可**：

| # | 条件 | 您的现状（实测） |
|---|---|---|
| 1 | 设备已绑定 + `auto_update = 1` | ✅ `auto_update = 1` |
| 2 | `ai_ota.type` == **设备上报的板型** | ❌ 现有记录 `type = bread-compact-esp32-lcd`，应为 `bread-compact-wifi-s3cam` |
| 3 | `ai_ota.version` **大于** 设备当前版本 | ⚠️ 记录为 `0.0.1`；设备 `app_version=2.5.2` → **不满足** |

> 注意第 3 条：设备上报的是**固件版本**（`app_version` 2.5.2），而 `assets.bin` 的"版本"复用同一字段比较。
> 若填 `0.0.1`，`compareVersions("0.0.1","2.5.2") < 0` → 不下发。**必须填大于 2.5.2 的版本号**（如 `2.5.3`）。

### 2. 回答：固件类型选什么？

**要选与设备上报板型完全一致的值：`bread-compact-wifi-s3cam`。**

但实测该值**不在** `FIRMWARE_TYPE` 字典中（字典 54 项，无 s3cam）。所以先补字典：

**方式 A：智控台 UI**
【字典管理】→ 找到 `FIRMWARE_TYPE` → 新增：
- 字典标签：`面包板 WiFi + Camera (ESP32-S3)`
- 字典值：`bread-compact-wifi-s3cam`
- 排序：`55`

**方式 B：直接入库**（`dict_type_id = 101`）

```sql
INSERT INTO sys_dict_data
  (dict_type_id, dict_label, dict_value, sort, remark, create_date)
VALUES
  (101, '面包板 WiFi + Camera (ESP32-S3)', 'bread-compact-wifi-s3cam', 55,
   'LifeOS 自建板型', NOW());
```

补完后刷新智控台，新建固件时即可选到该类型。

### 3. 完整操作步骤

**Step 1** 补字典（见上）

**Step 2** 在 generator 页面点「**下载 assets.bin**」（白色按钮，纯前端，不需要设备在线）

**Step 3** 智控台【OTA 固件管理】→ 新增：

| 字段 | 填什么 | 说明 |
|---|---|---|
| 固件类型 | `bread-compact-wifi-s3cam`（步骤 1 新增的） | **必须与设备上报板型一致** |
| 版本 | `2.5.3`（**大于设备当前 2.5.2**） | 否则 compareVersions 不通过 |
| 固件文件 | 刚下载的 `assets.bin` | 接受 `.bin/.apk/.wav` |
| 备注 | `assets theme v1` | 任意 |

**Step 4** 删除或改名旧的错误记录（避免 `getLatestOta` 取到它）：
```sql
DELETE FROM ai_ota WHERE type='bread-compact-esp32-lcd';
```
> `getLatestOta(type)` 是按 type 取最新；旧记录 type 不同，通常不影响，
> 但若曾把 type 改成 s3cam 则会被取到，需按版本比较。

**Step 5** 设备侧触发：
- 设备复位，或对它说触发重启；
- 设备启动 → POST `/xiaozhi/ota/`（携带它自己的 `board.type = bread-compact-wifi-s3cam`）
- 服务端命中记录且版本更高 → 返回 `firmware.url = http://192.168.1.3:8002/otaMag/download/<uuid>`
- 设备下载并替换 assets 分区 → 重启后生效

**Step 6** 验证（串口）：
```
I (xxxx) Ota: Current version: 2.5.2
I (xxxx) EspTcp: Resolved 192.168.1.3 -> 192.168.1.3
I (xxxx) HttpClient: Established new connection to 192.168.1.3:8002
I (xxxx) Ota: Activation successful / Ota: New version found: 2.5.3
```

### 4. 排障对照

| 现象 | 原因 |
|---|---|
| 设备 OTA 返回 `INVALID_FIRMWARE_URL` | ①`type` 不匹配 ②版本未大于当前 ③`auto_update=0` |
| 上传后设备无反应 | 检查 `ai_ota.type` 是否 == `ai_device.board` |
| 下载 404 | `download` 有 3 次重试上限（Redis），超限删除 uuid；需设备重新拉 OTA |
| 提示文件过大 | `assetsPartition.size` 小于 bin 大小，减小字体/表情 |
| `deviceReport.getBoard()` 为 null | 固件太旧未上报 board，先升级固件 |

### 5. 诚实边界

- 本条链路**只下发 assets 分区**（主题/表情/字体资源），**不是完整固件升级**；
- 若您要同时升级固件本身，需把 `type` 也匹配上并上传真正的 `xiaozhi.bin`，
  且设备端 OTA 处理逻辑要与分区布局一致；
- 全流程**不依赖 MQTT**（与附四的在线烧录不同），只依赖 HTTP OTA 接口，因此在本部署可用。

---

## 附六：摄像头/视觉功能失败 —— `'ascii' codec` 报错的真实含义

> 现象：让设备看画面，设备回复「镜头启动失败……传输管道被 ASCII 乱码堵住了」。
> 服务端日志反复出现 `'ascii' codec can't encode characters in position 7-8`。

### 1. 这个报错是假象（关键认知）

```
ERROR-配置错误: VLLM 的 API key 未设置,当前值为: 你的api_key
ERROR-Error in response generation: 'ascii' codec can't encode characters in position 7-8
ERROR-MCP Vision POST请求异常: 'ascii' codec can't encode characters in position 7-8
```

真实的错误是**第一行**：VLLM 的 api_key 仍是中文占位符 `你的api_key`。
openai SDK 把它写进 HTTP `Authorization` header 时，HTTP header 必须是 ASCII，
于是抛出编码异常——**后两行只是被编码异常掩盖的次生错误**。

`position 7-8` 恰好是 `你的api_key` 中文字符所在位置，可作为定位指纹。

**同一模式在本项目出现过两次**：
- LLM 调用失败（附三 §1.2）——LLM 的 key 是占位符
- 视觉调用失败（本节）——VLLM 的 key 是占位符

**通用结论**：凡见 `'ascii' codec can't encode characters in position N-M`，
先查**当前 provider 的 api_key 是否为中文占位符**，不要先怀疑网络或模型。

### 2. 完整调用链与失败点

```
设备："看看这是什么"
  → LLM 决策调用 self_camera_take_photo
  → MCP: self.camera.take_photo 下发到设备
  → 设备拍照，上传到 服务端 POST /mcp/vision/explain
  → Provider VLLM(openai 兼容) 初始化
     └─ 若 api_key 为占位符 → 编码崩溃 → 立即失败（本次）
     └─ 若 model_name 是纯文本模型 → 返回非图像理解结果
  → 结果回传 MCP → LLM 组织语言 → TTS 播报
```

### 3. 修复步骤（已实测）

**① 确认视觉模型配置**（`ai_model_config`，`model_type='VLLM'`）

```sql
SELECT model_code,
       JSON_UNQUOTE(JSON_EXTRACT(config_json,'$.model_name')) AS model_name,
       JSON_UNQUOTE(JSON_EXTRACT(config_json,'$.base_url'))   AS base_url,
       JSON_UNQUOTE(JSON_EXTRACT(config_json,'$.api_key'))    AS api_key
FROM ai_model_config WHERE model_type='VLLM';
```

要满足三点：
- `api_key` 是**真实 key**，不是 `你的api_key` / `??api_key`；
- `base_url` 指向支持**多模态**的端点（智谱：`https://open.bigmodel.cn/api/paas/v4/`）；
- `model_name` 是**多模态模型**（本次采用 `GLM-4.6V`）。

**② 一键修复脚本**（仓库内已提供）

```bash
# 把有效 LLM key 复制给 VLLM（仅当 VLLM 是占位符时）
docker cp scripts/fix_vllm_key.sql xiaozhi-esp32-server-db:/tmp/fix.sql
docker exec xiaozhi-esp32-server-db sh -c 'mysql -uroot -p123456 < /tmp/fix.sql'

# 设置多模态模型名
docker cp scripts/set_vllm_model.sql xiaozhi-esp32-server-db:/tmp/sv.sql
docker exec xiaozhi-esp32-server-db sh -c 'mysql -uroot -p123456 < /tmp/sv.sql'

# 重启生效
docker restart xiaozhi-esp32-server
```

**③ 用合成图验证模型（不依赖真机）**

```bash
docker cp scripts/probe_vllm.py xiaozhi-esp32-server:/tmp/probe_vllm.py
KEY=$(docker exec xiaozhi-esp32-server-db sh -c \
  "mysql -uroot -p123456 -N -e \"SELECT JSON_UNQUOTE(JSON_EXTRACT(config_json,'\$.api_key')) \
   FROM xiaozhi_esp32_server.ai_model_config WHERE model_code='ChatGLMVLLM' LIMIT 1;\"")

docker exec -e LLM_API_KEY="$KEY" -e VLLM_MODEL=GLM-4.6V \
  xiaozhi-esp32-server python /tmp/probe_vllm.py
```

实测输出：

```
OK HTTP 200  model=GLM-4.6V
REPLY: 白色
```

> `probe_vllm.py` 内置一张 1×1 PNG（Base64 data URI），**不需要真实照片**即可验证
> key + base_url + 模型名三者的组合是否可用。这是最省时间的排查手段。

### 4. 本次实测的模型可用性

| 模型名 | 结果 |
|---|---|
| `GLM-4.6V` | ✅ HTTP 200（采用） |
| `glm-4.6v` | ✅ HTTP 200（大小写不敏感） |
| `glm-4v-flash` | ✅ HTTP 200（旧版免费档，也可用） |

### 5. 验证成功判据

**服务端日志**（正常）：

```
INFO-执行工具: self_camera_take_photo，参数: {'question': '镜头前是什么？'}
INFO-发送客户端mcp工具调用请求: self.camera.take_photo
（无 配置错误 / 无 ascii 错误）
INFO-收到mcp消息：{... "text": "{\"success\":true, ...}"}
INFO-发送第一段语音: <画面描述>
```

**失败时的差异**：`"success": false` 且 message 含具体错误；
或根本没有 `self_camera_take_photo` 执行行（说明 LLM 没决定调用工具，是「意图识别」问题而非视觉问题）。

### 6. 排障速查

| 现象 | 首查 |
|---|---|
| `'ascii' codec ... position N-M` | 当前 provider 的 `api_key` 是否为中文占位符 |
| 「配置错误: XXX 的 API key 未设置」 | 该 provider 在智控台未填 key |
| 拍照成功但描述与画面无关 | `VLLM.model_name` 是文本模型而非多模态 |
| 无 `self_camera_take_photo` 执行记录 | LLM 未决定调用工具 → 检查意图/提示词，非视觉问题 |
| 视觉接口 401 | 直接 curl 该端点需认证 token；设备请求会带 token，属正常 |
| 照片上传失败 | 检查图片大小/格式与 `/mcp/vision/explain` 的入参契约 |

### 7. 诚实边界

- `GLM-4.6V` 的可用性仅验证了「能接受图片并返回文本」这一最小闭环；
  其描述质量、中文表达风格、对复杂场景的准确性**未做系统评测**；
- 视觉链路不影响文本对话主链路，二者 provider 独立配置。

