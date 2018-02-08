#### xhprof扩展安装
    git clone https://github.com/longxinH/xhprof
    cd xhprof/extension/
    phpize
    ./configure --with-php-config=/usr/local/php/bin/php-config --enable-xhprof
    make && make install
    
    php扩展配置
    [xhprof]
    extension=xhprof.so
    
#### mongodb安装
    centos
        tar zxf mongodb-linux-x86_64-3.2.9.tgz
        mv mongodb-linux-x86_64-3.2.9 /usr/local/mongodb
        cd /usr/local/mongodb
        mkdir db
    mac
        brew install mongodb
        
#### mongodb扩展安装
    
    #xhgui采用mongodb存储数据,首先需要安装mongodb

    tar zxf mongodb-1.3.0.tgz
    cd mongodb-1.3.0
    phpize
    mac
        ./configure --with-php-config=/usr/local/php/bin/php-config --with-openssl-dir=/usr/local/Cellar/openssl/1.0.2n
    centos
        ./configure --with-php-config=/usr/local/php/bin/php-config
    
    make && make install
    
    修改php.ini
        extension=mongodb.so
    
    重启php-fpm
    
#### mongodb 简单操作
    启动mongodb
        nohup /usr/local/mongodb/bin/mongod --dbpath /usr/local/mongodb/db & > /home/ding/log/mongodb_init.log
    
    客户端连接mongod
        #不指定--host就采用默认
        /usr/local/mongodb/bin/mongo --host 127.0.0.1:27017
        
        执行以下命令初始化xhprof数据库
        > use xhprof
        > db.results.ensureIndex( { 'meta.SERVER.REQUEST_TIME' : -1 } )
        > db.results.ensureIndex( { 'profile.main().wt' : -1 } )
        > db.results.ensureIndex( { 'profile.main().mu' : -1 } )
        > db.results.ensureIndex( { 'profile.main().cpu' : -1 } )
        > db.results.ensureIndex( { 'meta.url' : 1 } )
        > db.results.ensureIndex( { 'meta.simple_url' : 1 } )
        
        > exit
        
        清空数据
        db.results.remove({})
        
    客户端关闭服务器
            > use admin;
            > db.shutdownServer();
    
#### xhgui 图形界面安装配置

    代码安装
        cd ~/projects
        git clone git@github.com:dingyl/xhgui.git

    nginx配置
        xhgui.com
        root   /var/www/example.com/public/xhgui/webroot/;
        index  index.php;
        location / {
            try_files $uri $uri/ /index.php?$args;
        }
        include enable_php.config;
        
        
        
    在enable_php.conf中添加
        
        location ~ \.php {
            fastcgi_pass unix:/tmp/php-fpm.sock;
            fastcgi_index index.php;
            include fastcgi_params;
            # 最长执行时间
            fastcgi_read_timeout 300;
            fastcgi_split_path_info       ^(.+\.php)(/.+)$;
            fastcgi_param PATH_INFO       $fastcgi_path_info;
            #添加php预加载文件
            fastcgi_param PHP_VALUE "auto_prepend_file=/Users/apple/projects/xhgui/external/header.php";
            fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }