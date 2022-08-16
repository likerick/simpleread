> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Real_cTo/article/details/107305122?ops_request_misc=&request_id=&biz_id=102&utm_term=gitlab%20jenkins%20%E9%9B%86%E6%88%90%20%E9%92%89%E9%92%89&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-107305122.142^v41^pc_rank_34_queryrelevant25,185^v2^tag_show&spm=1018.2226.3001.4187)

### 实现功能：

*   代码提交 gitlab，自动触发 Jenkins 任务
*   Jenkins 任务完成后发送钉钉消息通知

#### [jenkins](https://so.csdn.net/so/search?q=jenkins&spm=1001.2101.3001.7020) 安装

下面是自动化安装 jenkins 的脚本：

```
#和jdk安装包在同一级目录下执行安装
#auth：    xinaho.zhang
#date：    2020.07.12
###################################################################################################3
VERSION=2.239

#java -version 
if [ ! -f /usr/bin/java ];then
  cd /usr/local/src
  yum install -y jdk-8u191-linux-x64.rpm &> /dev/null  
  touch /etc/profile.d/jdk.sh
cat > /etc/profile.d/jdk.sh << EOF
export JAVA_HOME=/usr/java/default
export PATH=$JAVA_HOME/bin:$PATH
EOF
. /etc/profile.d/jdk.sh
fi

wget https://mirror.tuna.tsinghua.edu.cn/jenkins/redhat/jenkins-${VERSION}-1.1.noarch.rpm
yum install jenkins-${VERSION}-1.1.noarch.rpm -y

systemctl start jenkins

cat > /var/lib/jenkins/hudson.model.UpdateCenter.xml << EOF
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json</url>
  </site>
</sites>
EOF
/usr/bin/sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /var/lib/jenkins/updates/default.json && /usr/bin/sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' /var/lib/jenkins/updates/default.json

systemctl restart jenkins

```

#### gitla 安装

1. 配置 [gitlab](https://so.csdn.net/so/search?q=gitlab&spm=1001.2101.3001.7020) 国内 yum 源安装 gitlab：

```
vim /etc/yum.repos.d/gitlab-ce.repo
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1

#清除并更新yum缓存
yum clean all && yum repolist


# 安装gitlab依赖软件
yum install curl policycoreutils openssh-server openssh-clients postfix


# 安装gitlab
yum install gitlab-ce  #安装gitlab，默认安装安装的为最新版本，安装时可以指定具体的版本进行安装。如果没有关闭防火墙需要配置防火墙

```

2. 配置启动 gitlab

```
#修改gitlab配置文件” /etc/gitlab/gitlab.rb”，绑定自己主机ip地址
# 将external修改为自己主机ip地址
external_url 'http://192.168.88.30'
# 启动gitlab并使修改的配置生效
gitlab-ctl reconfigure
#第一次启动配置可能需要好几分钟，建议gitlab主机配置内存2核4G以上

```

3. 登录 gitlab  
 **直接在浏览器中输入部署 gitlab 服务器的 iP 即可登录** (#80 端口是在初始化 gitlab 的时候启动的，因此如是之前有程序占用会导致初始化失败或无法访问)：  
![](https://img-blog.csdnimg.cn/20200712211203703.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
![](https://img-blog.csdnimg.cn/20200712211442516.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
gitlab 常用操作命令：

```
gitlab-ctl status：查看gitlab状态
gitlab-ctl start：启动gitlab
gitlab-ctl stop：停止gitlab
gitlab-ctl restart：重启gitlab
gitlab-ctl tail servername：查看gitlab集成的服务日志

```

#### 配置 jenkins 连接 gitlab

jenkins 安装常用插件： GitLab Plugin  
在系统管理 -> 系统配置里添加 gitlab 连接信息  
![](https://img-blog.csdnimg.cn/2020071221350568.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
其中 Credentials 选项添加 Gitlab API token 凭据：  
![](https://img-blog.csdnimg.cn/20200712213908169.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
![](https://img-blog.csdnimg.cn/20200712213831914.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
![](https://img-blog.csdnimg.cn/20200712213952797.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
![](https://img-blog.csdnimg.cn/20200712214103106.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)

#### 添加钉钉提示机器人

打开钉钉点击头像选择机器人管理：  
![](https://img-blog.csdnimg.cn/20200712214428831.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200712214653833.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200712214735508.png)  
完成后会生成一个 token 地址：  
![](https://img-blog.csdnimg.cn/20200712214836538.png)  
我们就可以在 jenkins 服务器上测试这个消息推送：

```
#其中access_token就是填我们在上部生成的token
curl 'https://oapi.dingtalk.com/robot/send?access_token=900f9db712ef5b75458f3bad6ca6ed1ebcc2ee1b64ae68693535d9af1ef28f79' \
   -H 'Content-Type: application/json' \
   -d '{"msgtype": "text","text": {"content": "测试警报,我就是我, 是不一样的烟火"}}'
#content里一定要写上我们自定义的关键词，否则报错

```

不出意外的话钉钉会收到提示消息  
![](https://img-blog.csdnimg.cn/2020071221520431.png)

#### 创建 jenkins job

在 gitlab 里新创建一个测试项目：  
![](https://img-blog.csdnimg.cn/20200712215801460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
然后我们通过我们的 IDE 或其他工具 push 一些文件上去：  
![](https://img-blog.csdnimg.cn/20200712220303298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
在 jenkins 里面创建一个 job：  
![](https://img-blog.csdnimg.cn/20200712220431279.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
然后我们选择构建触发器，并记录下 URL：  
![](https://img-blog.csdnimg.cn/20200712220526614.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
在 gitlab-setting 里选择 webhook：  
![](https://img-blog.csdnimg.cn/20200712220605517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
在 webhook 里 URL 填上次 jenkins 里记录的 URL，Token 在下面高级选项里生成  
![](https://img-blog.csdnimg.cn/20200712220746448.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/2020071222071327.png)  
添加 webhook：  
![](https://img-blog.csdnimg.cn/20200712221029212.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
然后在 jenkins job 里添加上我们的钉钉推送命令：  
![](https://img-blog.csdnimg.cn/20200712221433404.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)

然后我们保存 jenkins job，在 push 一些代码到 gitlab 上去测试：  
![](https://img-blog.csdnimg.cn/20200712221347600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JlYWxfY1Rv,size_16,color_FFFFFF,t_70)  
构建成功后：我们钉钉就会收到提示信息：  
![](https://img-blog.csdnimg.cn/20200712221709759.png)