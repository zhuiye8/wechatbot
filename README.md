<p align="center">
  <h2 align="center">WeChat Bot</h2>
  <p align="center">
    一款基于Wechaty的聊天机器人！
    <br/>
  </p>
</p>

<p align="center">
  <a href="https://github.com/lcjqyml/wechatbot/stargazers"><img src="https://img.shields.io/github/stars/lcjqyml/wechatbot?color=E2CDBC&amp;logo=github&amp;style=for-the-badge" alt="Github stars"></a>
  <a href="https://github.com/lcjqyml/wechatbot/actions/workflows/docker-latest.yml"><img src="https://img.shields.io/github/actions/workflow/status/lcjqyml/wechatbot/docker-latest.yml?color=E2CDBC&amp;logo=docker&amp;logoColor=white&amp;style=for-the-badge" alt="Docker build latest"></a>
  <a href="https://hub.docker.com/r/lcjqyml/wechatbot/"><img src="https://img.shields.io/docker/pulls/lcjqyml/wechatbot?color=E2CDBC&amp;logo=docker&amp;logoColor=white&amp;style=for-the-badge" alt="Docker Pulls"></a>
  <a href="./LICENSE"><img src="https://img.shields.io/github/license/lcjqyml/wechatbot?&amp;color=E2CDBC&amp;style=for-the-badge" alt="License"></a>
</p>

点击加入[qq交流群](http://qm.qq.com/cgi-bin/qm/qr?_wv=1027&k=xzKavu5qVsnb0Of5k0VBnFAKN1Knc46C&authKey=jQxOFByEpqVVA83Wj97kur4c0Ryzaz%2FRhQJyQu4JGR%2BqHF6zShTfxrf5OzUnLlgA&noverify=0&group_code=468682056)

## 配置说明

可以通过`config.yaml`或者环境变量进行配置：

| config.yaml | env | 说明 | 必须 | 默认 |
|----------|------|------|------|------|
| chatbotProxy | CHATBOT_PROXY | chatbot请求地址，参考[lss233/chatgpt-mirai-qq-bot][1] | YES | |
| autoAcceptFriendShip | AUTO_ACCEPT_FRIEND_SHIP | 自动通过好友请求 | NO | false | 
| autoAcceptRoomInvite | AUTO_ACCEPT_ROOM_INVITE | 自动通过群聊邀请 | NO | false | 
| privateChatTrigger | PRIVATE_CHAT_TRIGGER | 机器人私聊触发器，空则都触发 | NO | "" | 
| groupChatTrigger | GROUP_CHAT_TRIGGER | 机器人群聊触发器，空则@触发，否则：trigger 或 @bot trigger 触发 | NO | "" | 
| responseQuote | RESPONSE_QUOTE | 群聊中回复时是否引用触发的消息 | NO | false | 
| checkOnlineTrigger | CHECK_ONLINE_TRIGGER | 向发送此trigger的联系人定时反馈执行指令的结果 | NO | \_\_check\_\_ |
| checkOnlineCommand | CHECK_ONLINE_COMMAND | 定时执行的指令 | NO | ping |
| checkOnlineInterval | CHECK_ONLINE_INTERVAL | 定时间隔 | NO | 60 * 60 * 1000 |

### lss233/chatgpt-mirai-qq-bot 项目配置说明
本项目与lss233项目部署在同一服务器时：
<details>

* 为config.cfg增加以下配置：
    ```toml
    [http]
    host = "0.0.0.0"
    port = 8080
    debug = false
    ```
* cd至docker-compose.yaml目录，重启服务：
    ```bash
    docker-compose restart
    ```
* 查看日志，出现以下日志则表示http服务配置成功：
    ```bash
    docker-compose logs -f
    ```
    ```
    ......
    ... Running on http://0.0.0.0:8080 (CTRL + C to quit)
    ......
    ```
</details>

本项目与lss233项目部署在不同服务器时：
<details>
关键需要开放lss233服务的端口访问，以运行在云服务器为例：

* 为config.cfg增加以下配置：
  ```toml
  [http]
  host = "0.0.0.0"
  port = 8080
  debug = false
  ```
* cd至docker-compose.yaml目录，为docker-compose.yaml增加以下配置：
  ```yaml
  version: '3.4'
  services:
    ...
    chatgpt:
      image: lss233/chatgpt-mirai-qq-bot:browser-version
      ports:
        - "8080:8080"  # 关键是增加这个
      ...
  ```
* 重启服务docker-compose服务。
* 进入云服务商控制台，编辑安全组规则，入站规则增加8080端口。然后可以通过以下命令确认是否配置成功：
  ```bash
  # <server-ip>需要替换为部署机器人bot的服务器IP
  curl -i http://<server-ip>:8080/v1/chat -H 'Content-Type: application/json' -d '{"message":"ping"}'
  ```

</details>

## 运行说明
  
### docker部署

快速启动：
```bash
docker run -e CHATBOT_PROXY="<your-proxy>" lcjqyml/wechatbot:latest
# PS: linux服务器可执行此命令直接安装docker:
# sh <(curl -sL https://get.docker.com)
```
* `<your-proxy>` 需要替换为你自己搭建的http服务，例如：
    ```bash
    docker run -e CHATBOT_PROXY="http://127.0.0.1:8080" lcjqyml/wechatbot:latest
    ```
或者：
```bash
# <path-to-config> 需要替换为配置所在的实际路径
docker run -v /<path-to-config>/config.yaml:/app/config.yaml lcjqyml/wechatbot:latest
```
* 配置文件参考`config.yaml.example`

保存登陆记录，增加以下配置：
```bash
touch ./memory-card.json
docker run -v ./memory-card.json:/app/my-wechat-bot.memory-card.json ... lcjqyml/wechatbot:latest
```
遇到问题参考[ #30](https://github.com/lcjqyml/wechatbot/issues/30)

本项目与lss233项目部署在同一服务器时，执行以下命令：

```bash
docker run -e CHATBOT_PROXY="http://chatgpt-qq_chatgpt_1:8080" --network chatgpt-qq_default lcjqyml/wechatbot:latest
```
_PS: 以上chatgpt-qq_chatgpt_1, chatgpt-qq_default由docker-compose启动lss233项目时默认创建。_

启动后扫码登陆即可：
* 扫码的微信号需要进过实名认证，否则会异常。
* 尽量避免国外登陆或者异地登陆，防止封号。
* 若二维码不清晰，可将二维码上方的链接copy至浏览器打开扫码。

### Railway或者Zeabur托管部署
  
参考`issue` https://github.com/lcjqyml/wechatbot/issues/20

### 本地部署(linux x86)
安装nodejs、npm

下载压缩包：
```
curl -O https://nodejs.org/dist/v18.18.2/node-v18.18.2-linux-x64.tar.xz
```
解压压缩包：
```
tar xf node-v18.18.2-linux-x64.tar.xz
```
移动压缩包：
```
sudo mv node-v18.18.2-linux-x64 /usr/local/bin/
```
设置为全局命令：
```
sudo ln -s /usr/local/bin/node-v18.18.2-linux-x64/bin/node /usr/local/bin/node
sudo ln -s /usr/local/bin/node-v18.18.2-linux-x64/bin/npm /usr/local/bin/npm
```
如果现在使用```node -v```和```npm -v```命令可以正常输出版本信息，那么恭喜你，node和npm已经安装完成了，继续进行下面的步骤吧~

现在开始安装nodemon

npm更换国内源：（如服务器在国外可以跳过此步骤）
```
npm config set registry https://registry.npmmirror.com  
```
安装nodemon：
```
npm install -g nodemon
```
设置为全局命令：
```
sudo ln -s /usr/local/bin/node-v18.18.2-linux-x64/bin/nodemon /usr/bin/nodemon
```
拉取项目：
```
git clone https://github.com/lcjqyml/wechatbot.git
```
进入：
```
cd wechatbot
```
安装依赖：（如果安装失败，多重试几次）
```
npm install
```
设置环境变量：（如果使用```config.yml```进行配置，此处可掠过）
```
vim .env
```
填入环境变量：（如果使用```config.yml```进行配置，此处可掠过）
```
CHATBOT_PROXY="http://example.com:1234"
```
写一个小小的启动脚本，方便以后启动：
```
vim start.sh
```
填入内容：（如果使用```config.yml```进行配置，此处的```source ./.env```不需要添加）
```
source ./.env
nodemon
```
给予运行权限：
```
chmod +x start.sh
```
启动：
```
./start.sh
```
至此，linux本地部署完成，你可以把它放在一个screen窗口里运行，有关screen的用法请自行搜索


## 关联项目
* [lss233/chatgpt-mirai-qq-bot][1] - （本项目需要配合此项目的http service使用）多平台、多AI引擎的AIGC整合项目。
* [kx-Huang/ChatGPT-on-WeChat](https://github.com/kx-Huang/ChatGPT-on-WeChat) - 访问ChatGPT的微信聊天机器人


## 贡献者名单

欢迎提出新的点子、 Pull Request。  

<a href="https://github.com/lcjqyml/wechatbot/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=lcjqyml/wechatbot"  alt=""/>
</a>
Made with [contrib.rocks](https://contrib.rocks).

## 支持我们

如果这个项目对你有所帮助，请给我们一颗星。

<p align="center">
  <a href="https://github.com/lcjqyml/wechatbot/stargazers">
    <img src="https://reporoster.com/stars/dark/lcjqyml/wechatbot" alt="Stargazers repo roster for @lcjqyml/wechatbot" />
  </a>
  <a href="https://github.com/lcjqyml/wechatbot/stargazers">
    <img src="https://api.star-history.com/svg?repos=lcjqyml/wechatbot&type=Date" alt="Star history chart for @lcjqyml/wechatbot"/>
  </a>
</p>

[1]: https://github.com/lss233/chatgpt-mirai-qq-bot
