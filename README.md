# 自动化伪直播间开发日志
##### 罗智彬 2021年5月9日

## 需求分析
实现H5端伪直播带货，开发互动机器人营造万人在线的氛围
* 制作伪直播视频素材
* 实现后端推流和视频控制
* 实现H5互动直播间
* 直播前中后的后端控制
* 实现直播中带货点推送商品
* 实现实时互动机器人，与视频关键时间点契合
* 视频使用腾讯视频流量

## 技术&工具
* uniapp前端开发
* nvue（store状态保持：登录+websocket）
* video.js、video组件、jweixin-module腾讯视频加密
* docker-compose容器快速部署
* thinkphp6.0+thinkphp-swoole3
* docker-swoole
* PHP7.4
* Redis5.3.3
* MySQL
* swoole4.6.4
* 微信公众号支付+微信H5支付
* OBS工具（录播+推流）

## 伪直播解决方案
1. 制作录播视频，过程中有互动，有带货环节
> 问答预先策划好，后期关键帧插入对话
2. 视频伪装 vs 录像推流，方案二选一
> 视频隐藏控件实现视频伪装
> 使用腾讯云直播实现视频推流
3. 配合视频，多个时间点弹出商品，时间点后台可配置，也可以随时后端控制
> 后台可配置关键帧+话术内容脚本，购买链接、价格和封面
4. 客服机器人+真人在过程中进行监控、推广、互动
> 为每个用户分配fd，uid，为机器人分配一个或多个fd，通过数据结构协议推送
5. 实时的评论交互
> websocket+swoole通信技术实现
6. 其他直播间氛围效果
> 实时更新的在线人数、用户进入直播间消息，刷表情，提问，购课消息，礼物

## 实践
### 一、环境搭建
#### 所需物料
* docker
* docker-compose
* [thinkphp6.0+thinkphp-swoole3+websocket](https://gitee.com/tsbjt/thinkphp-swoole3-websocket/)
* [thinkphp6.0+thinkphp-swoole3扩展demo](https://gitee.com/tsbjt/thinkphp-swoole3-demo)
* [docker安装phpswoole/swoole](https://hub.docker.com/r/phpswoole/swoole)

#### 参考资料
* [supervisor配置文件(详细说明)](https://www.jianshu.com/p/7e788634257b)
* [Docker容器日志查看与清理的方式](https://www.cnblogs.com/aliases/p/13232384.html)

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
* 前端测试地址：https://yzws.eqscloud.com/static/web/ws.html
* wss连接地址：wss://yzws.eqscloud.com/ws?uid=1
* 测试数据包
	1. 发ping包
	```
	{"event":"ping","data":""}
	```
	2. 发点对点消息
	```
	{"event":"point","data":{"to":2,"content":"Hello"}}
	```
	3. 自定义事件
	* app/subscribe/WebSocketEvent.php文章中自定义事件rpa
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

### 二、直播视频
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

#### 详细步骤
1. 定义ws消息结构
> 根据互动直播间需求，定义几种消息结构，以实时控制直播间的各项功能
	1. 规定机器人的消息event类型为rpa
	2. 使用type区分不同控制功能，type对应参数如下
|type	|说明	|参数	|取值	|参数备注	|
|--	|--	|--	|--	|--	|
|1	|视频控制	|stage	|1直播前 2直播中 3直播后	|直播所在阶段	|
|	|	|src	|字符串	|视频地址	|
|	|	|start_time	|日期时间格式	|直播开始时间	|
|	|	|current_time	|整型	|当前时间点，单位秒	|
|2	|弹窗控制	|src	|字符串	|商品图片地址	|
|	|	|title	|字符串	|商品名称	|
|	|	|path	|字符串	|商品链接	|
|	|	|price	|整型	|现价，单位分	|
|	|	|original_price	|整型	|原价，单位分	|
|	|	|cd	|整型	|倒计时，单位秒	|
|	|	|position	|1左下排列 2商品图标	|出现位置和形式	|
|3	|评论	|msg_type	|1普通 2提问 3助教回答	|评论类型	|
|	|	|content	|字符串	|评论内容	|
|	|	|user_nickname	|字符串	|用户昵称	|
|	|	|uid	|整型	|用户id	|
|4	|当前在线人数	|num	|整型	|在线人数	|
|5	|新用户进入	|user_nickname	|字符串	|用户昵称	|
|6	|点赞	|num	|整型	|点赞数	|
|7	|其他控制	|control_type	|1清屏 2全体禁言 3单用户禁言	|直播中其他控制	|
|	|	|fd	|用户fd	|用户身份标识	|
|8	|礼物	|-	|-	|-	|
	3. 其中视频控制、弹窗控制、当前在线人数、点赞需要每次用户ws连接时推送最新值
	4. 配合前端做消息测试
	```
	//视频控制 type=1
	{
		"event":"rpa",
		"data":{
			"type":1,
			"stage":2,
			"src":"https://1255691551.vod2.myqcloud.com/81259e4fvodtransgzp1255691551/ea25c9ee5285890818160287740/v.f100030.mp4",
			"start_time":"2021-05-11 03:37:09",
			"current_time":20
		}
	}
	//弹窗控制 type=2
	{
		"event":"rpa",
		"data":{
			"type":2,
			"src":"https://xinli-1255691551.cos.ap-guangzhou.myqcloud.com/lessons/live/cover/210510.2.0.jpg",
			"title":"B高级睡眠质量提升课",
			"path":"",
			"price":99800,
			"original_price":199800,
			"cd":600,
			"position":1
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
	```

2. 初始化参数
	* 根据直播课ID，载入直播课具体参数，其中每场直播总时长=直播前互动时长+视频时长+直播后互动时长
	* 将直播课参数载入redis

3. 互动脚本配置
> 根据ws消息结构规范，制作互动脚本

4. 毫秒级定时任务
> 启动多个任务监听，根据互动脚本的时间点，取出任务数据，执行对应任务或任务组
	* 事件绑定/监听/订阅：配置app/event.php文件实现
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
	* \app\listener\SwooleTask
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
	
	* \app\listener\SwooleTaskFinish
	* \app\listener\SwooleBoot
	* \app\subscribe\Timer
	```
	// 同时监听多个定时任务
	```


### 四、后端业务开发
#### 数据字典
#### 详细步骤

### 五、前端开发
#### 参考资料
* [videojs 从上次播放的时间点开始播放](https://blog.csdn.net/qq285679784/article/details/103455097)
* [JS监听video视频播放时间](https://www.cnblogs.com/xxflz/p/10160879.html)
* [如何获取video.js中的当前播放时间](http://www.voidcn.com/article/p-oyvvdbpz-bue.html)

#### 详细步骤
