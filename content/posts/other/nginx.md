---
title: "Nginx"
date: 2021-01-04T11:31:30+08:00
author: ["loveyu"]
draft: falss
categories: 
- 其他
tags: 
- nginx
---



```sh
# 用于配置运行Nginx服务器的worker进程的用户和用户组
#user  nobody;

# 用于配置是否开启worker进程，默认是开启on
master_processes	off/on

# 配置worker进程的数量，数值越高nginx的并发处理量越大，一般为cpu的核心数，master_processes设置为off则这个属性无效
worker_processes  1;

# 是否以守护进程方式启动，守护进程就是是否随着终端关闭而关闭，默认为on
daemon	off/on

# 日志存储的文件路径
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

# pid的存储文件路径
#pid        /opt/homebrew/var/run/nginx.pid;

# 引入其他配置文件，可以在任何地方引入
include	xxx.conf

# 设置网络连接
events {
		# 设置网络连接的序列化，默认为on，为off的话当一个请求过来所有的worker都会去抢这个请求但只能有一个抢到其余没抢到的会休眠，设置为on会对多个nginx的worker进程进行序列化排号，只会一个一个唤醒来处理请求。on/off各有优缺点
		accpet_mutex on/off
		
		# 设置是否允许同时接受请求，默认为off
		mutil_accept on/off
		
		# 设置最大连接数，不能大于操作系统支持打开的最大句柄数
    worker_connections  1024;
    
    # 设置nginx选择那种事件驱动来处理网络请求，默认为epoll
    use		select/poll/epoll/kqueue
}


http {
		# 识别请求资源类型
    include       mime.types;
    # 设置nginx响应请求的mime类型
    default_type  application/octet-stream;

		# 设置日志的格式，main：格式名称，后边一堆是具体的日志格式组成
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

		# 设置日志位置 格式名称。main：上边声明的格式名称
    #access_log  logs/access.log  main;

		# 设置是否以nginx的sendfile()函数传输文件，改属性可以提高nginx处理静态资源的性能,默认off
		# 一下三个属性建议全on，新版本nginx可以兼容三个属性全开
    sendfile        on;
    
    # 该配置必须在sendfile开启的前提下才能使用，只要是用来提升网络包的传输效率，默认off 
    tcp_nopush     on;
    
    # 该配置必须在keepalive连接开启的前提下使用，来提高网络包传输的实时性，默认on
		tcp_nodeplay 	on;

    # 设置nginx长连接的超时时间，格式为秒
    keepalive_timeout  65;
    
    # 设置keep_alive连接使用次数
    keepalive_requests 100;
    
    # 开启/关闭压缩功能，默认off
    gzip  on;
    # 压缩什么类型的文件，gzip开启这个属性也要开启
    gzip_types application/javascript text/html text/css application/json;
    # 全部都压缩，不建议
    gzip_types *;
    # 压缩等级，默认为1，数字越大压缩越高效率越低
    gzip_comp_level 1-9
    # 设置相应头携带Vary：Accept-encoding，作用是告诉接收方所发送的数据经过了gzip的压缩处理
    gzip_vary		on;
    # 设置处理压缩的缓冲区数量大小，格式为：number size。默认为32 4k ｜ 16 8k
    # number是指定nginx服务器向系统申请缓存空间的个数，size值得是每个缓存空间的大小
    # 该属性默认即可无需设置
    gzip_buffers	32 4k
    # 针对不同种类的客户端发起的请求选择性的开启和关闭gzip功能
    # 这里的表达式会和请求头的user-agent的值做对比
    gzip_disable	正则表达式
    # 针对http协议版本选择性的开启或者关闭gzip，默认1.1
    gzip_http_version		1.0 | 1.1
    # 针对传输数据的大小来选择行的开启或者关闭gzip，默认为20，单位默认bytes字节，可以设置为kb和m
    # 对比的值是请求头content-length的值，比这个值大就不会压缩了
    gzip_min_length		20

		# 设置服务
		# 当一个请求能被多个server匹配到时，匹配顺序为：精准匹配、通配符在开始时、通配符在结束时、正则表达式、default_server、第一个声明的server
    server {
    		# 设置监听的端口，可以是ip:端口，也可以是只有端口，还可以只是ip，只是ip呢就是监听该ip的全部端口
    		# 在没有设置default_server属性时，当请求无法在配置中找到的时候默认会响应第一个声明的server配置
    		# lister 8080 default_server	当设置default_server属性时，无法匹配的请求则会去找设置了default_server属性的server配置
        listen       8080;
        # 设置虚拟主机的服务名称，多个名称则空格分割；设置方式有：精确匹配、通配符匹配、正则表达式匹配；通配符支持’*‘，但’*‘只能用到首尾部分，不能在中间使用。例如www.a*b.com是不行的，只能*.abc.com、www.abc.*	
        # 正则表达式必须是~开头，例如: ~^www\.(\w+)\.com$
        # 可以是ip、localhost、域名
        server_name  localhost;
        
        # 设置请求的url
        # url设置为/符号开头的，则所有以/对应值开头的都能访问到。例如设置为/abc则/abc、/abcde、/abc/ 这些都会匹配到
        # 以=符号开头的时精确匹配，例如=/abc只能匹配/abc请求，连/abc/都无法匹配到
        # 以～开头表示当前url中包含了正则达表示，并且区分大小写
        # 以～*开头的表示当前url中包含了正则表达式，并且不区分大小写
        # 以^开头的和无符号开头的功能是一样的，但是加了^的location被匹配到时就不会继续往下找其他location了，例如^/abcd，当请求为/abcd时匹配到了^/abcd的location时就不会继续找其他匹配的location了
        # 多个location匹配，优先匹配的是最后一个匹配的location
        location / {
        		default_type text/html;
        		# 这里的$1就可以获取正则表达式中的对应括号值，$1就是获取到第一个匹配的括号
        		# 例如：www.abc.com 这里的$1获取的就是abc
        		return 200 '$1'
        }
        # root的处理结果是：root路径+location路径
        root 目录
        # alias的处理结果是：使用alias路径替换location路径
        # alias设置的目录可以是虚拟的但root设置的目录是必须存在的
        alias 目录
        
        # 例如：要访问这个图片html/images/a.png。当这个请求设置为/images/的时候，alias的路径最后也必须加上/，例如：alias html/images/。root不需要
        location /images {
        		# 设置 root html 请求为；/images/a.png是可以访问到的
        		root html
        		# 设置 alias html 请求为/images/a.png是访问不到的，原因就是alias是把/images/a.png替换为html/a.png所以访问不到，要想访问到就要设置为：alias html/images  这样/images/a.png就可以访问到
        		alias html
        		
        		# index 设置默认首页，值可以为多个，请求会一次先后匹配直到有该文件为止
        		# 请求为 /images 返回的为a.png这个图片，不需要/images/a.png就可以访问到（前提alias设置为html/images）
        		index a.png
        		
        		# 设置错误页面，设置格式为 状态码 指定资源，可以是静态资源也可以是一个域名
        		err_page	404 err.html
        }
        
        # 设置拦截的请求
        location /get_test {
        		# 设置返回类型
            default_type    text/html;
            # 设置返回值
            return  200 "get_test";
        }
        location /get_json {
        		# 设置返回为json
            default_type    application/json;
            return  200 "{\"name\":\"zhangsan\",\"age\":12}";
        }

        #charset koi8-r;
				
        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            # index 设置默认首页，值可以为多个，请求会一次先后匹配直到有该文件为止
            index  index.html index.htm;
        }

				# 设置响应状态码对应的静态页面
        #error_page  404              /404.html;
        
        # 重定向到/50x.html请求
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        # 使用@符号来配置
        err_page 404 @jump_to_error
        location @jump_to_error {
        		# 对应配置
        		....
        }
        
        # 修改状态码，当找不到页面的时候不会返回404状态码而是返回200状态码
        # 格式必须是 被修改状态码空格等号要修改成的状态码
        error_page 404 =200 /50x.html
        ocation /50x.html {
        	....
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
    include servers/*;
}

```
