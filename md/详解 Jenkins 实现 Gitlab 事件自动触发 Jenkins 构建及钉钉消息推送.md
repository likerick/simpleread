> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6954602475676663839)

实践环境
====

*   GitLab Community Edition 12.6.4
*   Jenkins 2.284
*   Post build task 1.9（Jenkins 插件）
*   Generic Webhook Trigger Plugin 1.72（Jenkins 插件）
*   GitLab 1.5.13（Jenkins 插件

实现步骤
====

钉钉机器人配置
-------

1.  选择要推送的钉钉群
2.  点击群设置按钮
3.  点击智能群助手
4.  点击添加机器人
5.  点击添加机器人 + 号按钮
6.  点击自定义
7.  填写机器人名字，用于匹配推送消息请求体内容的的关键词

### 截图如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb2a6a1b85f342ccbe213c266320e3f7~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 复制出 Webhook 地址，供下文钉钉消息推送 Shell 脚本中使用，完成

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93bd10e022bd4645bf67e0ec56bafaff~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 安装 Jenkins 插件新建并配置 Jenkins 项目 Build Triggers 配置如下，勾选 Generic Webhook Trigger

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17b40455cbc6487ab57ba8f3f044fffb~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### Post content parameters(因为 Gitlab 触发的请求为 post 请求，需要基于请求体内容来判断是否执行 Jenkins 构建) 关键配置项说明：

Variable 自定义变量名称

Expression 用于提取变量值的表达式 (支持 JSONPath、XPath)，提取的值赋值给上述自定义变量 (例中为 event_name)。

Option Filter 关键配置项说明：

Expression 用于匹配下述 Text 的正则表达式，如果匹配则执行构建请求，否则不执行。这里配置为 ^push$，是因为 Gitlab merge 合并代码操作触发的请求，其请求体为 json 格式数据，其中包含名为 event_name 的键，其值为 push

Text 用于匹配上述正则表达式的文本，例中设置为自定义变量 $event_name。

以上配置大意为，如果收到构建请求，使用 JSONPath 表达式从 JSON 格式的请求体获取键为 event_name 的值，存储到名为 event_name 变量，然后取该变量值同正则表达式 ^push$ 匹配，如果匹配，则触发 Jenkins 构建当前项目，否则不构建。

Token：自定义 token 值，用于请求 http://JENKINS_URL/generic-webhook-trigger/invoke 触发构建使用，如下，可以用于查询参数、请求头参数

/invoke?token=TOKEN_HEREtoken: TOKEN_HEREAuthorization: Bearer TOKEN_HERE generic-webhook-trigger 配置参考连接

[plugins.jenkins.io/generic-web…](https://link.juejin.cn?target=https%3A%2F%2Fplugins.jenkins.io%2Fgeneric-webhook-trigger%2F "https://plugins.jenkins.io/generic-webhook-trigger/") Post-build Actions 配置

点击 Add post-build action 按钮，弹出界面中选择 Post build task 可新增以下配置界面。如下，可在 Script 输入框中编写构建完成后需要执行的 Shell 命令 (该插件会先根据填写的 shell 命令生成一个临时 sh 脚本，然后执行该脚本)，例中为钉钉推送命令，具体代码参见下文

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dcf56578882048229158e400c1c76ecb~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp) 如上图，如果只希望构建成功才执行 Script，可以勾选 Run script only if all previous steps were successful 钉钉消息推送 Shell

```
#!/bin/bash
#################################################################
# 作者：shouke
# 日期：2021-03-07
# 作用：机器人通知
#################################################################
# 钉钉消息变量定义
#################################################################
# 当前时间
TIME_NOW=`date "+%Y年%m月%d日 %H:%M:%S"`
BUILD_STATUS="失败"
LAST_BUILD_BUILD_XML=`curl http://ops.dev.xxxx.com/view/testarch/job/$JOB_NAME/lastBuild/api/xml --user juser_name:123456`
BUILD_RESULT=$(echo $LAST_BUILD_BUILD_XML | grep "<result>SUCCESS</result>") 
if [ "${BUILD_RESULT}" ];then 
    BUILD_STATUS="成功"
else
    BUILD_RESULT=$(echo $LAST_BUILD_BUILD_XML | grep "<result>FAILURE</result>") 
    if [ "${BUILD_RESULT}" ];then 
        BUILD_STATUS="失败"
    else
        BUILD_STATUS="无法获取"     
    fi
fi   
# 机器人 webhook 地址(上文添加钉钉机器人结束时复制的webhook地址)
DINGTALK_WEBHOOK_URL='https://oapi.dingtalk.com/robot/send?access_token=903fcd6c56f301d0a57bee243792a11bb1e42cae89af5a9071bdba890c0a3d2'
# 消息标题 # 实际不起作用，但是不能少，否则发送失败
DINGTALK_TITLE="XX平台有新的构建，请及时查阅"
# 消息正文
# Jenkins Job构建日志地址
JENKINS_JOB_BUILD_LOG_URL="http://ops.dev.xxxx.com/view/testarch/job/${JOB_NAME}/${BUILD_NUMBER}/console"
DINGTALK_TEXT="## xx平台有新的构建，请及时查阅\n\n>\
**【通知时间】**：${TIME_NOW}\n\n>\
**【构建ID】**：${BUILD_DISPLAY_NAME}\n\n>\
**【构建项目】**：${JOB_NAME}\n\n>\
**【构建状态】**：${BUILD_STATUS}\n\n>\
**[点击查看更多](${JENKINS_JOB_BUILD_LOG_URL})**\n
" 
#  
# 发送钉钉消息通知函数
#################################################################
function SEND_MESSAGE_TO_DINGTALK() {
    /usr/bin/curl "$1" -H 'Content-Type: application/json' -d "
    {
        \"markdown\": {
            \"title\": \"$2\", 
            \"text\": \"$3\"
        }, 
        \"at\": {
          \"atMobiles\": [],
          \"isAtAll\": false
        },
        \"msgtype\": \"markdown\"
    }
    " 
}
# 发送钉钉消息
#################################################################
SEND_MESSAGE_TO_DINGTALK "${DINGTALK_WEBHOOK_URL}" "${DINGTALK_TITLE}" "${DINGTALK_TEXT}"
复制代码

```

说明： curl [ops.dev.xxxx.com/view/testar…](https://link.juejin.cn?target=http%3A%2F%2Fops.dev.xxxx.com%2Fview%2Ftestarch%2Fjob%2F%24JOB_NAME%2FlastBuild%2Fapi%2Fxml "http://ops.dev.xxxx.com/view/testarch/job/$JOB_NAME/lastBuild/api/xml") --user juser_name:123456

一名为 juser_name 的用户，使用密码 123456 访问指定项目的最后一次构建相关的信息，返回 xml 文档

注意：钉钉聊天窗口中要实现消息换行必须使用两个 \ n

Gitlab 自动触发配置 Settings -> Integration，打开如下页面，

*   填写 URL([ops.dev.xxxx.com/generic-web…](https://link.juejin.cn?target=http%3A%2F%2Fops.dev.xxxx.com%2Fgeneric-webhook-trigger%2Finvoke%3Ftoken%3D0771826b93bbd566266bce34f5123ebb)%25EF%25BC%258C%25E8%25BF%2599%25E9%2587%258C%25E7%259A%2584token%25E5%2580%25BC%25E5%258D%25B3%25E4%25B8%25BAgeneric-webhook-trigger%25E6%258F%2592%25E4%25BB%25B6%25E4%25B8%25AD%25E9%2585%258D%25E7%25BD%25AE%25E5%259C%25A8%25E5%25AE%259A%25E4%25B9%2589token%25E5%2580%25BC "http://ops.dev.xxxx.com/generic-webhook-trigger/invoke?token=0771826b93bbd566266bce34f5123ebb)%EF%BC%8C%E8%BF%99%E9%87%8C%E7%9A%84token%E5%80%BC%E5%8D%B3%E4%B8%BAgeneric-webhook-trigger%E6%8F%92%E4%BB%B6%E4%B8%AD%E9%85%8D%E7%BD%AE%E5%9C%A8%E5%AE%9A%E4%B9%89token%E5%80%BC")
*   勾选 Push events 触发器 (这里以 push、合并代码操作为例子，所以仅勾选该事件)
*   勾选 Enable SSL verification 复选框 (如果没有勾选的话，默认就是勾选的) 最后点击 Add webhook 按钮

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9831283eb6994dcb9a8ffb29c376903b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp) 添加的配置，会自动显示在下方，可以对其进行事件触发测试

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4358dd3e754e4b39b7128dafeffd0cbb~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp) 触发的记录会自动在配置编辑页面下方显示，点击 View details 按钮，可以查看请求明细:

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20168dbd1f0e46a1a234362d17ea6e97~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp) 注意：自动触发时 Jenkins 项目构建时，如果 Jenkins 使用了参数化构建插件 Build With Parameters Plugin，并且使用插件实现的参数有设置默认值，则自动触发时也会自动使用对应参数的默认值进行构建。 钉钉消息推送效果图:

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/73c8ade62a8546699fe1961e92277e03~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

我的博客即将同步至腾讯云 + 社区，邀请大家一同入驻：[cloud.tencent.com/developer/s…](https://link.juejin.cn?target=https%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Fsupport-plan%3Finvite_code%3D3euvsjtqoegwc "https://cloud.tencent.com/developer/support-plan?invite_code=3euvsjtqoegwc")