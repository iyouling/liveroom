# 自动化伪直播间开发文档
##### 罗智彬 2021年5月9日

[自动化伪直播Demo地址](https://xinli.yoga2018.cn/xl/live1.html)

## 需求分析
实现H5端伪直播带货，开发互动机器人营造万人在线的氛围
- 制作伪直播视频素材
- 实现后端推流和视频控制
- 实现H5互动直播间
- 直播前中后各阶段的通信控制
- 实现直播中带货点推送商品
- 实现实时互动机器人，与视频关键时间点契合
- 使用腾讯视频优化视频流量成本


## 技术&工具
- uniapp前端开发
- nvue（store状态保持：登录+websocket）
- video.js、video组件、jweixin-module腾讯视频加密
- docker-compose容器编排快速部署
- docker-swoole
- thinkphp6.0+thinkphp-swoole3
- PHP7.4
- Redis5.3.3
- MySQL
- swoole4.6.4
- 微信公众号支付+微信H5支付
- OBS工具（录播+推流）


## 伪直播解决方案
1. 制作录播视频，过程中有互动，有带货环节
> 问答预先策划好，后期关键帧插入对话

2. 视频伪装 vs 录像推流，方案二选一
> 视频隐藏控件实现视频伪装
> 
> 使用腾讯云直播实现视频推流

3. 配合视频，多个时间点弹出商品，时间点后台可配置，也可以随时后端控制
> 后台可配置关键帧+话术内容脚本，购买链接、价格和封面

4. 客服机器人+真人在过程中进行监控、推广、互动
> 为每个用户分配fd，uid，为机器人分配一个或多个fd，通过数据结构协议推送

5. 实时的评论交互
> swoole3+websocket通信技术实现

6. 直播间氛围效果
> 实时更新的在线人数、用户进入直播间消息，刷表情，提问，购课消息，礼物


## 较佳实践
### 一、环境搭建
#### 所需物料
- docker
- docker-compose
- [thinkphp6.0+thinkphp-swoole3+websocket](https://gitee.com/tsbjt/thinkphp-swoole3-websocket/)
- [thinkphp6.0+thinkphp-swoole3扩展demo](https://gitee.com/tsbjt/thinkphp-swoole3-demo)
- [docker安装phpswoole/swoole](https://hub.docker.com/r/phpswoole/swoole)

#### 参考资料
- [supervisor配置文件(详细说明)](https://www.jianshu.com/p/7e788634257b)
- [Docker容器日志查看与清理的方式](https://www.cnblogs.com/aliases/p/13232384.html)

#### 详细步骤
1. 安装docker

2. 安装docker-compose

3. 域名解析+SSL证书

4. 上传thinkphp-swoole3-demo源码

5. 上传nginx/conf及ssl证书文件

6. docker-compose.yml文件制作

7. swoole容器内操作
> 容器内通过supervisor管理thinkphp-swoole服务
```
docker exec -it eqs_swoole sh //进入容器
supervisorctl status //查看服务状态
supervisorctl reload //服务热更新
supervisorctl restart all //重启全部服务
```

#### 环境搭建完成后测试
- 前端测试地址：https://yzws.xinli.link/static/web/ws.html

- wss连接地址：wss://yzws.xinli.link/ws?uid=1

- 测试数据包
1. 发ping包
```
{"event":"ping","data":""}
```

2. 发点对点消息
```
{"event":"point","data":{"to":2,"content":"Hello"}}
```

3. 自定义事件
* app/subscribe/WebSocketEvent.php文件中自定义事件rpa
```
public function onRpa($event)
{
	foreach ($this->table->u2fd as $row) {
		$this->server->push($row['fd'], json_encode([
			'type' => $event['type'] ?? 0,
			'msg' => $event['msg'] ?? "hello",
			'online_count' => time(),
			'sender' => "rpa"
		], 320));
	}
}
```

* 发包测试
```
{"event":"rpa","data":{"type":2,"msg":"Hello World"}}
```


### 二、直播视频准备
#### OBS推流工具

#### 无他相机开启虚拟摄像头

#### 腾讯云直播服务

#### 腾讯云点播服务

#### 直播录制
1. 录制准备

2. 录制前

3. 录制中

4. 录制后

5. 录制相关问题


### 三、机器人自动化
> RPA (Robotic Process Automation)

#### 参考资料
[Swoole定时队列任务+消息推送](https://www.cnblogs.com/zhengweizhao/p/10410511.html)

#### 核心技术分析
1. 视频伪装 vs 录像推流
- 伪直播的视频播放有两种方案：一是录像推流的传统方式，二是播放视频文件伪装成直播推流。

- 录像推流：使用OBS推流工具进行推流。
	* 优点：视频与OBS推流保持同步播放，H5有10秒左右延迟，不需要通过程序控制播放时间的同步。
	* 缺点：需要使用电脑设备保持运行；不适合连续复播的场景；无法节省视频流量费用。
	* 步骤：先用OBS等工具录制视频并完成后期，再通过腾讯云直播配置好推流域名和播放域名，最后通过OBS进行推流。
	* 适用场景：单场直播、直播视频画面可控，可随时中止。

- 视频伪装：通过程序控制视频的播放时间点，达到直播的效果。
	* 难点：视频播放时间点的同步，包括当前直播状态（前中后）、当前直播时间点计算、与客户端通信同步
	* 优点：不需要使用设备进行推流，使用腾讯云点播存储视频，可优化流量费用，可进行连续重复直播。
	* 缺点：需要通过后端算法保证视频的同步；如果要中止直播，需要通过程序控制；mp4视频会被探测并下载。
	* 步骤：先用OBS等工具录制视频并完成后期，再上传云存储，最后通过程序控制视频的播放同步。
	* 适用场景：连续多场直播

- H5前端播放视频统一使用videojs组件，可同时适配视频mp4格式和m3u8推流格式

2. 通信技术
> 采用swoole3+websocket通信框架
> 
> 单独架设服务器，与业务后端分离，避免直播业务单体故障影响其他业务。
>
> 环境搭建、技术实现、通信测试已在一、环境搭建中详述

3. 同步算法
> 视频伪装和录像推流两种方案都要解决消息通信同步问题
> 
> 采用视频伪装的方案，还需解决视频播放同步问题

- 1 视频播放同步

	|空窗期	|1轮直播前	|1轮直播中	|1轮直播后	|2轮直播前	|2轮直播中	|2轮直播后	|...	|
	|--	|--	|--	|--	|--	|--	|--	|--	|
	|stage4	|stage1	|stage2	|stage3	|stage1	|stage2	|stage3	|...	|

	* 设定直播时间由四部分组成：空窗期（第一次直播时间在未来）+直播前时长+直播时长（一般为视频时长）+直播后时长
	* 对于连续直播场景：第一次直播后，不再有空窗期，只由单轮直播时长=直播前时长+直播时长+直播后时长组成
	* 为了控制整体同步调度，定义life_time参数表示单轮直播中，当前时间所处的本轮直播时间点位置，仅在空窗期取值为负数，其余时期0≤本参数≤单轮直播时长
	* 为了控制视频播放时间点同步（视频伪装方案），定义current_time参数表示单轮直播中，当前时间所处的视频播放的时间点位置，在空窗期和直播前取值为负数
	* 定义关键时间参数如下

	|参数	|说明	|示例	|数据定义	|
	|--	|--	|--	|--	|
	|stage	|直播所处阶段	|1前2中3后4空窗期	|算法见下	|
	|first_time	|第一次直播时间点（日期时间）	|2021-05-13 11:35:00	|数据库字段	|
	|live_start_at	|直播开始前时长（秒）	|120	|数据库字段	|
	|live_duration	|直播持续时长，一般为视频时长（秒）	|3323	|数据库字段	|
	|live_end_last	|直播结束后时长（秒）	|300	|数据库字段	|
	|first_timestamp	|第一次直播中开始时间戳	|1620876900	|strtotime(first_time)	|
	|first_start_timestamp	|第一次直播前开始时间戳	|1620876780	|first_timestamp-live_start_at	|
	|window_time	|空窗期时长（秒）	|10000	|first_start_timestamp-time()	|
	|live_free_time	|两轮直播间隙时长（秒）	|420	|live_start_at+live_end_last	|
	|round_time	|单轮直播平均时长（秒）	|3743	|live_free_time+live_duration	|
	|life_time	|当前时间所处的本轮直播时间点位置（秒）	|-100	|算法见下	|
	|current_time	|当前时间视频的播放时间点（秒）	|-220	|life_time-live_start_at	|
	|start_timestamp	|下次直播开始时间戳	|1620886900	|算法见下	|
	|start_time	|下次直播开始点（日期时间）	|2021-05-13 14:21:40	|date(start_timestamp)	|

```
/*
* $life_time 当前时间所处的本轮直播时间点位置
* $stage 直播所处阶段
* $start_timestamp 下次直播开始时间戳
*/
$time = time();
if($window_time > 0)
{
	$stage = 4; //当前时间为空窗期
	$start_timestamp = $first_timestamp; //下次直播开始时间为第一次直播开始时间
	$life_time = -$window_time; //单轮时间点为负值
	$current_time = $life_time - $live_start_at; //当轮视频播放时间点为负值
}
else
{
	$life_time = ($time - $first_start_timestamp) % $round_time;
	$current_time = $life_time - $live_start_at; //当轮视频播放时间点为负值
	if($current_time < 0)
	{
		$start_timestamp = $time - $current_time;
		$stage = 1; //直播即将开始
	}
	else
	{
		$start_timestamp = $time - $current_time + $round_time;
		if($current_time <= $live_duration)
		{
			$stage = 2; //正在直播
		}
		else
		{
			$stage = 3; //直播已结束
		}
	}
}
```

- 2 直播间整体同步调度



4. 后端控制

5. 前端控制



#### 详细步骤
1. 定义ws消息结构
> 根据互动直播间需求，定义几种消息结构，以实时控制直播间的各项功能

- 规定机器人的消息event类型为rpa
- 使用type区分不同控制功能，type对应参数如下

|type	|说明	|参数	|取值	|参数备注	|
|--	|--	|--	|--	|--	|
|1	|视频控制	|stage	|1直播前 2直播中 3直播后	|直播所在阶段	|
|	|	|src	|字符串	|视频地址	|
|	|	|start_time	|日期时间格式	|直播开始时间	|
|	|	|current_time	|整型	|当前时间点，单位秒	|
|	|	|is_chat_forbidden	|bool	|是否全局禁言	|
|2	|弹窗控制	|src	|字符串	|商品图片地址	|
|	|	|document_id	|整型	|商品ID	|
|	|	|title	|字符串	|商品名称	|
|	|	|path	|字符串	|商品链接	|
|	|	|width	|整型	|图片长度upx	|
|	|	|height	|整型	|图片宽度upx	|
|	|	|close_top	|整型	|关闭按钮位置upx	|
|	|	|close_right	|整型	|关闭按钮位置upx	|
|	|	|close_show	|bool	|是否显示按钮	|
|	|	|price	|整型	|现价，单位分	|
|	|	|original_price	|整型	|原价，单位分	|
|	|	|cd	|整型	|倒计时，单位秒	|
|	|	|position	|整型	|出现位置次序	|
|	|	|close	|bool	|是否关闭弹窗	|
|3	|评论	|msg_type	|1普通 2提问 3助教回答	|评论类型	|
|	|	|content	|字符串	|评论内容	|
|	|	|user_nickname	|字符串	|用户昵称	|
|	|	|uid	|整型	|用户id	|
|4	|当前在线人数	|num	|整型	|在线人数	|
|5	|提示信息	|chat_info	|字符串	|提示信息	|
|6	|点赞	|num	|整型	|点赞数	|
|7	|其他控制	|control_type	|1全体禁言 2单用户禁言 3清屏	|直播中其他控制	|
|	|	|uid	|用户uid	|用户uid	|
|	|	|is_forbidden	|true禁言 false解禁	|是否禁言	|
|8	|直播同步	|-	|-	|-	|
|9	|礼物	|-	|-	|-	|

- 其中视频控制、弹窗控制、当前在线人数、点赞需要每次用户ws连接时推送最新值
- 配合前端做消息测试
```
//视频控制 type=1
{
	"event":"rpa",
	"data":{
		"type":1,
		"stage":2,
		"src":"https://1255691551.vod2.myqcloud.com/81259e4fvodtransgzp1255691551/ea25c9ee5285890818160287740/v.f100030.mp4",
		"start_time":"2021-05-11 03:37:09",
		"current_time":20,
		"is_chat_forbidden":false
	}
}
//弹窗控制 type=2
{
  "event": "rpa",
  "data": {
    "type": 2,
    "document_id": 13,
    "src": "https://xinli-1255691551.cos.ap-guangzhou.myqcloud.com/live/e2.png",
    "width": 500,
    "height": 164,
    "title": "高级睡眠质量提升课程",
    "path": "",
    "price": 69900,
    "original_price": 79900,
    "cd": 600,
    "position": 2,
    "close_show": false,
    "close_top": 28,
    "close_right": 22,
    "close": false
  }
}
//评论 type=3
{
	"event":"rpa",
	"data":{
		"type":3,
		"msg_type":1,
		"content":"老师讲得真好",
		"user_nickname":"小花",
		"uid":888
	}
}
//其他控制 type=7
{
	"event":"rpa",
	"data":{
		"type":7,
		"control_type":2,
		"uid":1022,
		"is_forbidden":true
	}
}
```

2. 初始化参数
- 根据直播课ID，载入直播课具体参数，其中每场直播总时长=直播前互动时长+视频时长+直播后互动时长
- 将直播课参数载入redis

3. 互动脚本配置
> 根据ws消息结构规范，制作互动脚本
> 
[评论消息生成](https://www.xinli.link/index.php?s=/admin/stat/gene_live_msg.html)

4. 毫秒级定时任务
> 启动多个任务监听，根据互动脚本的时间点，取出任务数据，执行对应任务或任务组
- 事件绑定/监听/订阅：配置app/event.php文件实现
```
//事件定义文件
//无绑定事件，有监听swoole启动，task任务投递，task任务完成三个事件，有订阅定时器事件
return [
    'bind' => [
    ],
    'listen' => [
        'AppInit' => [],
        'HttpRun' => [],
        'HttpEnd' => [],
        'LogLevel' => [],
        'LogWrite' => [],
        'swoole.task' => ['\app\listener\SwooleTask'],
        'swoole.finish' => ['\app\listener\SwooleTaskFinish'],
        //init中无法使用swoole_timer_tick等函数
        'swoole.init' => ['\app\listener\SwooleBoot'],
        //managerStart中无法使用addProcess
		//'swoole.managerStart' => ['\app\listener\SwooleBoot'],
    ],
    'subscribe' => [
        '\app\subscribe\Timer'
    ],
];
```

- \app\listener\SwooleTask
```
// 当监听到Swoole的$server->task投递任务，执行程序
// 实现了可配置的任务处理，会检查config/task.php配置里是否有配置相应的执行命令
public function __construct(Server $server, Container $container)
{
	$this->server = $server;
	$this->table = $container->get(Table::class);
	$this->key = config('task.key');
	$this->alias = config('task.alias');
}
public function handle(Task $task)
{
	if (isset($task->data[$this->key]) && isset($this->alias[$task->data[$this->key]])) {
		$class = new $this->alias[$task->data[$this->key]]['class'];
		$methods = $this->alias[$task->data[$this->key]]['methods'];
		foreach ($methods as $method) {
			$class->$method($task->data);
		}
		if ($this->alias[$task->data[$this->key]]['finish']) {
			$task->finish($task->data);
		}
	} else {
		dump($task->data);
		dump('未定义的task任务:' . json_encode($task->data, 320));
	}
	return;
}
```
	
- \app\listener\SwooleTaskFinish
- \app\listener\SwooleBoot
- \app\subscribe\Timer
```
// 同时监听多个定时任务
```


### 四、前端开发
#### 参考资料
- [videojs 从上次播放的时间点开始播放](https://blog.csdn.net/qq285679784/article/details/103455097)
- [JS监听video视频播放时间](https://www.cnblogs.com/xxflz/p/10160879.html)
- [如何获取video.js中的当前播放时间](http://www.voidcn.com/article/p-oyvvdbpz-bue.html)

#### 详细步骤
- 表情包
```
"[微笑]", "[色]", "[发呆]", "[抽烟]", "[抠鼻]", "[哭]", "[发怒]",
"[呲牙]", "[睡]", "[害羞]", "[调皮]", "[晕]", "[衰]", "[闭嘴]",
"[指点]", "[关注]", "[搞定]", "[胜利]", "[无奈]", "[打脸]", "[大笑]",
"[哈欠]", "[害怕]", "[喜欢]", "[困]", "[疑问]", "[伤心]", "[鼓掌]",
"[得意]", "[捂嘴]", "[惊恐]", "[思考]", "[吐血]", "[卖萌]", "[嘘]",
"[生气]", "[尴尬]", "[笑哭]", "[口罩]", "[斜眼]", "[酷]", "[脸红]",
"[大叫]", "[眼泪]", "[见钱]", "[嘟]", "[吓]", "[开心]", "[想哭]",
"[郁闷]", "[互粉]", "[赞]", "[拜托]", "[唇]", "[粉]", "[666]",
"[玫瑰]", "[黄瓜]", "[啤酒]", "[无语]", "[纠结]", "[吐舌]", "[差评]",
"[飞吻]", "[再见]", "[拒绝]", "[耳机]", "[抱抱]", "[嘴]", "[露牙]",
"[黄狗]", "[灰狗]", "[蓝狗]", "[狗]", "[脸黑]", "[吃瓜]", "[绿帽]",
"[汗]", "[摸头]", "[阴险]", "[擦汗]", "[瞪眼]", "[疼]", "[鬼脸]",
"[拇指]", "[亲]", "[大吐]", "[高兴]", "[敲打]", "[加油]", "[吐]",
"[握手]", "[18禁]", "[菜刀]", "[威武]", "[给力]", "[爱心]", "[心碎]",
"[便便]", "[礼物]", "[生日]", "[喝彩]", "[雷]"
```

- scroll-with-animation 卡顿问题


### 四、后端业务开发
#### 数据字典

#### 详细步骤
大数计算bcdiv需安装bcmath扩展
[安装包](http://rpmfind.net/linux/rpm2html/search.php?query=php-bcmath)
deb10u1 选择 Fedora 34 for s390x
wget http://rpmfind.net/linux/fedora-secondary/releases/34/Everything/s390x/os/Packages/p/php-bcmath-7.4.16-1.fc34.s390x.rpm /
解压命令：rpm2cpio php-bcmath-7.4.16-1.fc34.s390x.rpm | cpio -div
cp /etc/php.d/20-bcmath.ini /usr/local/etc/php/conf.d/20-bcmath.ini
cp /usr/lib64/php/modules/bcmath.so /usr/local/lib/php/extensions/no-debug-non-zts-20190902 && ldconfig
