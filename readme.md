# Emby Pinyin

此工具能使emby或jellyfin支持电影、电视剧和音乐的拼音首字母排序和搜索，如果觉得此工具帮到你，可以点个Star⭐️！

## 目录
- [Emby Pinyin](#emby-pinyin)
  - [目录](#目录)
  - [特性](#特性)
  - [已支持的内容类型](#已支持的内容类型)
  - [使用方法](#使用方法)
    - [windows系统使用方法](#windows系统使用方法)
    - [windows、linux及mac等系统通用php环境使用方法](#windowslinux及mac等系统通用php环境使用方法)
    - [Docker](#docker)
    - [Docker Webhook and Cron](#docker-webhook-and-cron)
    - [拼音排序方式](#拼音排序方式)
    - [拼音搜索](#拼音搜索)
    - [参数化执行](#参数化执行)
    - [Webhooks Server](#webhooks-server)
  - [运行截图](#运行截图)
  - [使用效果](#使用效果)

## 特性
- 自动保存历史服务器配置，方便下次执行
- 一键处理所有媒体库或自定义媒体库
- 同时支持emby和jellyfin

## 已支持的内容类型

- [x] 电影
- [x] 音乐
- [x] 电视节目
- [ ] 有声读物
- [ ] 书籍
- [ ] 游戏
- [ ] 音乐视频
- [x] 家庭视频与照片
- [ ] 混合内容

## 使用方法

### windows系统使用方法

为方便没有windows php环境的用户，直接打包了exe程序执行：

1. 下载：[release](https://github.com/hisune/emby_pinyin/releases)
2. 解压后打开文件夹里面的EmbyPinyin.exe
3. 输入你的emby或jellyfin服务器地址、API密钥（首次输入后，下次可不用输入，直接选择）
4. 选择排序方式及要处理的媒体库（如不确定此工具对你的媒体库产生的影响，可自行选择一个媒体库处理，确认没问题后再处理所有媒体库，而不是一开始就处理所有媒体库）
5. 等待处理完成

### windows、linux及mac等系统通用php环境使用方法

> 需要`PHP7.2`及以上版本环境，并修改`php.ini`的`phar.readonly`配置项为`Off`

首次执行：

```sh
composer create-project hisune/emby_pinyin
cd emby_pinyin
composer pre-install
composer start
```

二次执行：
```sh
composer start
```

执行完毕以上命令后的操作步骤和windows版本一致

### Docker

Build image:

```sh
docker build -t emby_pinyin .
```

运行：

```sh
docker run --rm -it emby_pinyin
```

### Docker Webhook and Cron

镜像支持监听webhook服务，或通过crond定时执行排序

Build image:

```sh
docker build -t emby_pinyin.webhook -f Dockerfile.webhook .
```

环境变量：

|ENV|默认值|说明|
|-|-|-|
|WEBHOOK_ENABLED|0|是否开启webhook，默认关闭|
|CRON_ENABLED|0|是否开启cron，默认关闭|
|CRON_SCHEDULE|0 * * * *|cron执行周期，默认每个整点执行|
|HOST|http://example:8096|jellyfin的host，必须改为自己的地址|
|API_KEY|*****|jellyfin的key，必须改为自己的值|
|SORT_TYPE|1|排序方式，参考[参数化执行](#参数化执行)|

webhook监听80端口，请自行映射到host

```sh
docker run -d \
    -e WEBHOOK_ENABLED=1 \
    -e CRON_ENABLED=1 \
    -e CRON_SCHEDULE="0 * * * *" \
    -e HOST="http://example:8096" \
    -e API_KEY="*****" \
    -e SORT_TYPE=1 \
    -p 8080:80 \
    --name emby_pinyin emby_pinyin.webhook
```

执行`docker logs -f emby_pinyin`以及`docker exec -it emby_pinyin tail -f /var/log/cron`可查看cron执行日志

### 拼音排序方式
以“测试”俩字为例，不同排序方式的最终结果如下(默认为首字母方式)：
1. 首字母：cs
2. 全拼：ceshi
3. 前置字母：c测试
4. emby默认：测试

### 拼音搜索
指定originaltitle参数，将修改OriginalTitle（即原标题）字段，从而实现拼音搜索。 originaltitle参数的取值和type参数一致，当传入值`4`时，会将OriginalTitle修改为和标题相同的值。

> 注意：originaltitle参数修改后无法还原为原系统默认值，如你需要保留你媒体的原标题，请谨慎修改。

### 参数化执行
通过对程序传入一定的固定参数能实现：
- 配合计划任务执行实现自动化的拼音转换功能（例如每隔n小时自动执行转换拼音任务）
- 可以实现快速命令执行，例如每次运行的时候将server参数和type参数进行固定，而无需执行程序后手动选取

当前可用参数：

| 参数全称          | 参数缩写 | 说明                                           |
|---------------|------|----------------------------------------------|
| server        | s    | 服务器编号                                        |
| host          | h    | 指定服务器地址，配合key参数使用，例如：http://192.168.1.1:8096 |
| key           | k    | 指定服务器API密钥，配合host参数使用                        |
| type          | t    | 排序方式，1：首字母，2：全拼，3：前置字母，4：服务器默认               |
| all           | a    | 是否处理所有媒体库，y是，n否                              |
| media         | m    | 媒体库编号                                        |
| originaltitle | o    | 修改OriginalTitle的方式，值参考type参数，默认不修改           |
| help          | H    | 获取帮助                                         |

windows exe举例：

> windows可以使用创建快捷方式或创建bat脚本的方式传递参数值

```shell
# 参数缩写方式举例
.\EmbyPinyin.exe -s 1 -t 3 -a n -m 2
```

linux&mac&windows等系统通用php环境举例：

```shell
# 参数全称方式举例
comopser start -- --server=1 --type=3 --all=n --media=2
```

### Webhooks Server
Emby Server从4.7.9.0开始支持“新媒体已添加”的webhook事件，emby_pinyin也从1.0.0版本开始支持，使用webhooks server功能能实现添加媒体库内容后自动执行新内容的拼音排序，做到无人值守，无需手动运行。

> 推荐在linux环境下执行webhooks server

1. 开启服务：通过以下命令开启webhooks server:
    ```shell
    # 安装emby_pinyin
    composer create-project hisune/emby_pinyin
    cd emby_pinyin
    composer pre-install
    # 启动http服务
    php -S localhost:9091
    ```
    如果你想监听局域网请求，可以将localhost换成当前执行命令的本机局域网ip，另外监听端口9091也可以自定义。
2. 确定请求参数：如果你执行且保存过服务器信息，使用server参数即可，例如：`?server=1`；你也可以直接使用host和key参数，指定服务器信息，例如：`?host=192.168.1.168&key=服务器API密钥`。两种方式必选一种。
3. emby设置Webhooks：打开emby管理后台，定位到`服务器`->`Webhooks`->`添加Webhooks`，输入自定义名称，url填写`http://localhost:9091/run.php`和第2步的请求参数组装的字符串，例如：`http://localhost:9091/run.php?server=1`
4. jellyfin设置Webhooks：打开jellyfin管理后台定位到`控制台`->`插件`，安装weebhook插件，点击`Webhooks`->`Add Generic Destination`，输入自名称及url同emby;`Notification Type`选择`Item Added`，勾选`Send All Properties`

推荐使用supervisor来管理你的webhooks server，安装supervisor的方法以centos为例：
```shell
# 安装supervisor
yum update -y
yum -y install epel-release
yum -y install supervisor
systemctl start supervisord
systemctl enable supervisord
# 配置supervisor
cd /etc/supervisord.d
vim emby_pinyin.ini
```
示例ini配置如下，请自行修改emby_pinyin所在路径，及监听的IP和端口：
```ini
[program:emby_pinyin]
command=/usr/bin/php -S localhost:9091
stderr_logfile=/home/wwwroot/emby_pinyin/err.log
stdout_logfile=/home/wwwroot/emby_pinyin/out.log
directory=/home/wwwroot/emby_pinyin/
autostart=true
user=root
```
最后启动emby_pinyin：
```shell
supervisorctl update
# 查看状态
supervisorctl status
```

## 贡献

感谢以下的小伙伴们对于 emby_pinyin 项目的代码贡献，让这个项目变得越来越好！

[![Contributors](https://opencollective.com/emby_pinyin/contributors.svg?width=890&button=false)](https://github.com/hisune/emby_pinyin/graphs/contributors)

## 运行截图

![](https://raw.githubusercontent.com/hisune/images/master/emby_pinyin_2.jpg)


## 使用效果

![](https://raw.githubusercontent.com/hisune/images/master/emby_pinyin_1.jpg)

## Sponsor
Thanks to [zmto](https://www.zmto.com/) for providing VPS sponsorship.
