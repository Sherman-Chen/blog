# nginx上配置php调试器xdebug

## 一、前言
所涉及到的技术有nginx、php、xdebug、php-dev（用于php扩展的编译）、vim和vim插件vdebug。

php-dev在php包中就有，如果没有的话，也可以使用yum来安装：

```
# yum search php|grep dev

php55w-devel.x86_64 : Files needed for building PHP extensions devices
php56w-devel.x86_64 : Files needed for building PHP extensions devices
php70w-devel.x86_64 : Files needed for building PHP extensions devices
php71w-devel.x86_64 : Files needed for building PHP extensions devices

```

上面找到多个版本的，比如55就是5.5了，我服务器上是php7.0的，因此选择php70w安装。

而vdebug则可以直接从github上面中clone到本地，比如

```
git clone https://github.com/joonty/vdebug.git
```

我机子上使用的是https协议，但是本文讨论的方法对http协议是没有影响的。

## 二、安装xdebug

xdebug官方提供了分析工具，网址为https://xdebug.org/wizard.php。步骤为：

1. 在服务器配置一个脚本(phpinfo.php)，用来输出php的配置信息：
```
<?php
echo phpinfo();
```

之后在浏览器中访问该文件，复制phpinfo的信息到上面的网站中，然后点击网站下方的按钮进行分析，之后xdebug便会提供详细的步骤进行配置xdebug。

> 记得如果在生产环境，删除phpinfo.php文件。



配置xdebug：
1. 在phpinfo中查看配置文件的路径
2. 编辑php.ini
3. 加入xdebug的路径zend_extension = /usr/lib64/php/modules/xdebug.so
4. 编辑其他字段，比如：
    xdebug.remote_port
    xdebug.remote_autostart
5. 配置安装好了之后，再次运行phpinfo文件，就可以在里面查看到xdebug的信息了
6. 重启php-fpm，service php-fpm restart


## 三、配置vim插件

将vdebug插件clone到本地之后，只需要将里面的plugin文件夹和doc复制到~/.vim文件夹中即可。
```
cp -R xxx ~/.vim/
```
xxx分别为plugin和doc目录，因此需要使用R递归选项。

配置好了之后，打开vim，按下F5，如果出现正在监听信息，说明配置成功。

可以在~/.vimrc文件中，配置vdebug的信息：
```
let g:vdebug_options = {
    \ 'debug_file': 'vdebug_log',
    \ 'debug_file_level': 2,
    \ 'watch_window_style': 'expanded',
    \ 'marker_default': '*',
    \ 'continuous_mode': 0,
    \ 'ide_key': '',
    \ 'break_on_open': 1,
    \ 'on_close': 'detach',
    \ 'path_maps': {'/var/www': '~/code/xdebug-test'},
    \ 'marker_closed_tree': '+',
    \ 'timeout': 20,
    \ 'port': 9001,
    \ 'marker_open_tree': '-',
    \ 'debug_window_level': 1,
    \ 'server': '127.0.0.1' }
```

之后在要断点的地方按下F10下断点，之后按下F5监听，在浏览器上运行的路径加上?XDEBUG_SESSION_START=1，就能够断点调试了。
> 按下F5之后，需要再20秒内运行对应的网址

> php配置信息加上xdebug.remote_autostart=1，运行的网址后面就不需要加上?XDEBUG_SESSION_START=1了。但是由于我这边测试机子开发人员还需要从接口上获取信息，因此我的路径是没有配置remote_autostart的。

## 四、常见问题

1. vdebug监听很快就关闭了。
nginx+php-fpm中，php的fastcgi是监听在9000端口的，而xdebug和vdebug也是默认监听9000端口，因此需要将xdebug的端口跟vdebug的端口改变。

xdebug配置：
```
xdebug.remote_enable=1
xdebug.remote_port=9001
xdebug.remote_autostart=1
```

~/.vimrc配置

```
let g:vdebug_options = {
    \ 'port': 9001,
    \ 'server': '127.0.0.1' }
```

2. 504 Gateway Time-out

由于我们调试的时候，php是没有响应nginx的，因此超时会造成nginx返回504。我们可以nginx的配置文件中新增：

```
fastcgi_connect_timeout 连接超时，只有连接了我们才有断点，因此不用配置
fastcgi_send_timeout 两次send的超时，也使用默认
fastcgi_read_timeout 两次read，是这个了
```

因此在nginx中新增：
```
fastcgi_read_timeout 500s;
```
比如：
```
server{
    fastcgi_read_timeout 500s;
    ...
    location /{
        ...
    }
}
```
之后重启nginx服务，比如`service nginx restart`

> 请谨慎，以上配置都是只适用于测试机，不适用于生成机。

3. 配置快捷键

还是编辑~/.vimrc:

```
let g:vdebug_keymap = {
    \    "run" : "<F5>",
    \    "run_to_cursor" : "<F9>",
    \    "step_over" : "<F2>",
    \    "step_into" : "<F3>",
    \    "step_out" : "<F4>",
    \    "close" : "<F6>",
    \    "detach" : "<F7>",
    \    "set_breakpoint" : "<F10>",
    \    "get_context" : "<F11>",
    \    "eval_under_cursor" : "<F12>",
    \    "eval_visual" : "<Leader>e",
    \}
```




