# timePHP

##timePHP用途？
timePHP是一个基于php cli开发的定时脚本框架,可以实现简单的配置,自己的逻辑代码纯php无需写shell脚本
易管理,易开发。简单的配置一下就可以根据需求开发自己的逻辑代码。

##timePHP操作命令
##全部启动命令
```
[root@iZbp1if228spaovivfbbyjZ CronTab]# php ./start.php start all &
[启动成功]
```
##单一启动命令
```
[root@iZbp1if228spaovivfbbyjZ CronTab]# php ./start.php start backup
[启动成功]
```
###关闭命令
```
[root@iZbp1if228spaovivfbbyjZ CronTab]# php ./start.php kill
[关闭成功]
```
###单一关闭命令
```
[root@iZbp1if228spaovivfbbyjZ CronTab]# php ./start.php kill backup
[关闭成功]
```
###查看命令
```
[root@iZbp1if228spaovivfbbyjZ CronTab]# php ./start.php select

----------------------- timePHP -----------------------------
timePHP version:1.0          PHP version:5.6.22
------------------------ timePHP -------------------------------
pid           name          status
1853          clearroom       [OK] 
1854          backup          [OK] 
1855          clearmeesage    [OK] 
-----------------------------------------------------------

```
##目录结构
```
./task							任务目录
./task/crontab					任务组
./task/crontab/backup			任务目录
./task/extend					第三方插件目录
./task/common.php				方法
./task/config.php				配置文件
./timePHP						框架核心代码
./start.php						启动文件
	
```
###任务代码
例:
```
<?php
/**
 * 备份数据库
 * @author Administrator
 *  */
namespace Crontab;
import("PHPMailer.PHPMailerAutoload");//邮件发送插件
class init{
    public static function _init(){
        $date = date("Ymd");
        $sql_arr=array();
        foreach(C("TASK.backup")['dbArray'] as $db_name){
            $sql_name=C("TASK.backup")['target_dir'].$db_name."_".$date.".sql";
            $command ="mysqldump -u ".C("DB.DB_USER")." -p".replace_keyword(C("DB.DB_PWD"))." ".$db_name." >".$sql_name;
            shell_exec($command);
            $sql_arr[]=$sql_name;
        }
        //邮件发送C("EMAIL.GET_EMAIL")
        timePHP_send_mail(C("EMAIL.GET_EMAIL"),'测试备份-'.date("Y-m-d H:i:s"),'备份测试环境-'.date("Y-m-d H:i:s",time()),'<p>邮件来了测试</p>',$sql_arr);
        //删除 过期的数据库备份数据
        $past_time=C("TASK.backup")['target_dir'];
        for($i=$past_time;$i>=1;$i--){
            foreach(C("TASK.backup")['db_array'] as $db_name){
                $command="rm -rf ".C("TASK.backup")['target_dir'].$db_name."_".date("Ymd",time()-(($i+$past_time)*86400)).".sql";
                shell_exec($command);//是否删除 过期的数据库
            }
        }
    }
}
```
##配置文件规范
```
例:
<?php 
/**
 * 任务进程中的配置文件
 * time 单位秒
 * @var array  */
return [
    //任务 id
    "TASK"=>[
        "clearmeesage"=>[
            "time"=>60,
            "number"=>0,
            "name"=>"clearmeesage"
        ],//清除短信中不要的垃圾数据
        "clearroom"=>[
            "time"=>3600,
            "number"=>1,
            "name"=>"clearroom"
        ],//清除房间中不要的垃圾数据
        "backup"=>[
            "time"=>86400,//多久备份一次
            "number"=>2,
            "name"=>"backup",
            "target_dir"=>"/home/bak/",//备份的路径
            "dbArray"=>[//要备份的数据库
                "tourism_game",
            ],
            "past_time"=>9//过期时间/天
        ],//数据库定时备份
    ],
    "EXECUTE"=>["clearroom","backup","clearmeesage"],//需要启动的任务 "backup",
    //数据库信息
    "DB"=>array(
        'DB_TYPE' => 'mysql',
        'DB_HOST' => 'localhost',
        'DB_NAME' => '',//数据库名称
        'DB_USER' => '',//数据库账号
        'DB_PWD' => '',//密码
        'DB_PORT' => '3306',
        'DB_CODE'=>'utf8'
    ),
    //邮件发送配置
    "EMAIL"=>[
        'SMTP_HOST'   => 'smtp.qq.com', //SMTP服务器
        'SMTP_PORT'   => '587', //SMTP服务器端口
        'SMTP_USER'   => '8044023@qq.com', //SMTP服务器用户名
        'SMTP_PASS'   => '', //SMTP服务器密码
        'FROM_EMAIL'  => '8044023@qq.com', //发件人EMAIL
        'FROM_NAME'   => '测试附件服务器端', //发件人名称
        'REPLY_EMAIL' => '', //回复EMAIL（留空则为发件人EMAIL）
        'REPLY_NAME'  => '', //回复名称（留空则为发件人名称）
        "GET_EMAIL"   => 'cqkxm@qq.com',//接收邮箱地址
    ],
];
?>
```
##数据库操作
###查询单条
```
M("User")->where("id=1")->find();
```
###查询多条
```
M("User")->where("id=1")->select();
```
###删除一条
```
M("User")->where("id=1")->delete();
```
###获取总条数
```
M("User")->where("id=1")->count();
```
###修改
```
$data=array(
    'username'=>'小张',
    'password'=>md5(123456)	
);
M("User")->where("uid=1")->save($data);
```
###新增
```
$data=array(
    'username'=>'小张',
    'password'=>md5(123456)	
);
M("User")->add($data);//返回单条新增id
```
###筛选字段
```
M("User")->field('username')->where("id=1")->find();
M("User")->field('username')->where("id=1")->select();
```
###查询复杂的sql语句
$sql="select * from a left join b on a.id=b.id";
```
M()->execute($sql);
```
##公共函数
###获取配置信息
```
C('DB.type')
```
###打印
```
dump()
```
###替换特殊字符串
```
replace_keyword（$str）
```
##日志
###写入日志
```
Log::write($log);
```
日志路径:timePHP/error.log。
###自定义日志
```
"LOG"=>[//日志相关配置
        "log_path"=>"timePHP/",
        "log_file_size"=>2097152,
        "log_time_format"=>"c",
        "log_name"=>""
    ]
```
###日志格式
```
[警告][2017-01-20 16:19:51]\nFILE:D:\phpStudy\wwwroot\www.work.com\timePHP\lib\time\Course.php
LINE:27行
FUNCTION:run
LEVEL:2
CODE:703
MSG:你的操作命令错误
-------------------------------------------------------------------------------------------------
```

