# laravel-email-queue

php版本：7.1.3
laravel版本：5.8.*

第一步：
配置.env文件
```
QUEUE_DRIVER=database
MAIL_DRIVER=smtp
MAIL_HOST=smtp.163.com
MAIL_PORT=25
MAIL_USERNAME=[username@email.com]
MAIL_PASSWORD=[password]
MAIL_ENCRYPTION=null
```
php artisan make:model Post -m -c
创建发送email的模型和创建对应数据库的migrate

第二步：
编辑database/migrations/2019_**_**_******_create_posts_table.php
在up方法下写入：
```
Schema::create('posts', function (Blueprint $table) {
      $table->bigIncrements('id');
      $table->string('title');
      $table->string('body');
      $table->timestamps();
  });
```
用来创建posts数据表
执行：php artisan migrate（此时，查看数据表是否多了一个posts）

第三步：
编辑路由文件和控制器对应的方法
控制器：
```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Jobs\SendReminderEmail;
use App\Post;

class PostController extends Controller
{
    public function store(Request $request)
    {
        $post = new Post;
        $post->title = $request->title;
        $post->body = $request->body;

        $post->save();
        $this->dispatch(new SendReminderEmail($post)); // 队列
    }
}
```

第四步：
运行：php artisan make:job SendReminderEmail 
生成job类，编辑
```
<?php
namespace App\Jobs;

use App\Post;
use Illuminate\Support\Facades\Mail;
use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;

class SendReminderEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    private $post;

    public function __construct(Post $post)
    {
        $this->post = $post;
    }

    public function handle()
    {
        $data= array(
            'title'=> $this->post->title,
            'body'=> $this->post->body,
        );
        // emails.post 对应的视图文件模板
        Mail::send('emails.post', $data, function($message){
            $message->from('wei****@163.com', 'Hi Queues');//发送邮箱，发件人昵称
            $message->to('731*****@qq.com')->subject('There is title');//邮件title
        });
        echo 'email send success';die;
    }
}
```

第五步：
创建模板文件（定义发邮件的文本格式）
resources/views/emails/post.blade.php
此处两个变量，一定要和上一步$data中的变量一样
```
<p> A new post has been created </p>
<p> {!! $title !!}</p>
<p> {!! $body  !!} </p>
```

第六步：
执行：php artisan queue:work监听队列任务

最后：
call controller中的function，把title和body参数带上就可以收到邮件了！
