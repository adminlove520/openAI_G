### 最近持续优化中，喜欢的好兄弟给个🌟关注一下

### 一、介绍
- 能干什么？ 
  - 通过调用`OpenAI`的接口智能回答问题。
  - 可以直接api调用
  - 可以用作公众号自动回复。（提示：仅用于个人娱乐体验，不适合商业用途）
- 是`ChatGPT`吗？  不是。`ChatGPT`基于GPT-3.5，本项目是调用GPT-3，有非常大差距。现在`ChatGPT`还没开放接口，安全限制很高，现有市面都是GPT-3。
- 免费吗？不算，`OpenAI`账号赠送18$，限期使用。 消耗根据问题和回复长度计算。
- 有什么不足？ 
  - 回复内容准确度仅供参考，更适合开放性问题。 
  - 不支持上下文。 (也方便做，但是花费更多的tokens）
  - 速度和回复长度很难兼得。如果是订阅号，只能被动回复，[限制最久15s做出回复](https://developers.weixin.qq.com/doc/offiaccount/Message_Management/Passive_user_reply_message.html)，回复可能超时或者是截断的(做了缓存优化，可稍等再次提问直接获得答案)  。 非订阅号，48小时内有20条主动回复额度，这个版本也开发了，在dev分支，还不稳定。 
- 风险。依托于公众号存在一定风险。 我已加了[敏感词检测](https://github.com/tomatocuke/sieve) ，但不清楚微信具体机制此举是否有效。
- 体验。关注公众号`风中玲音`尝试提问，这仅是个人娱乐号，不推送。


### 公众号部署
> 如果仅用于api调用，忽略下文有关微信公众号的设置、忽略 WX_TOKEN 参数
1. 获取`API_KEY`。[OpenAI](https://beta.openai.com/account/api-keys) （如果访问被拒绝，注意全局代理，打开调试，Application清除LocalStorage后刷新，实测可以）
2. 获取微信公众号`令牌Token`：[微信公众平台](https://mp.weixin.qq.com/)->基本配置->服务器配置->令牌(Token)  (不使用公众号可调过)
3. 使用以上参数启动服务，以下两种方式选其一部署。(此处举例端口9001，如果用公众号且无域名须用80端口)
  - Docker
    ```bash
    docker run -p 9001:8080 -e API_KEY=xxx -e WX_TOKEN=xxx -d -v $PWD/log:/app/log tomatocuke/openai
    ```
  - Golang
    ```bash 
    git clone https://github.com/tomatocuke/openai.git
    cd openai
    go run main.go -PORT=9001 -API_KEY=xxx -WX_TOKEN=xxx 
    ```
4. 启动服务后简单测试 `curl 'http://127.0.0.1:9001/test?msg=中国在哪个洲'` 
5. 查看日志 `tail ./log/data.log`
6. 公众号配置。 
  - 无域名。须用80端口部署，服务器地址(URL)填写 `http://服务器IP/wx`。
  - 有域名。nginx配置参考
    ```conf
    server {
      listen 80;
      server_name 域名; #你的域名，不带http，例: abc.com

      location /openai/ {
        proxy_pass http://127.0.0.1:9001/; # 服务端口号
      }
    }
    ```
    重新加载nginx配置`nginx -s reload`后，公众号服务器地址填写: `http://域名/openai/wx`。
    启用公众号服务器配置  (初次设置可能要等待2分钟生效）
7. 直接调用api `curl 'http://域名/openai/test?msg=中国在哪个洲'`
    

### 三、其他
- 2023年3月6日，官方api报错，猜测目标对国内进行限制。
日志取证：
  ```
发生错误「Post "https://api.openai.com/v1/completions": dial tcp 31.13.91.6:443: i/o timeout」，您重试一下 
  ```
- 2023年3月8日00:43:53，代理报错：
- 发生错误「Post "https://api.openai.com/v1/chat/completions": proxyconnect tcp: EOF」，您重试一下
- 目测是代理的问题，暂时更换代理方式  
- 
