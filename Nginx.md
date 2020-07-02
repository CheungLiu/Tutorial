## Nginx

1. 安装pcre和nginx

   https://www.runoob.com/linux/nginx-install-setup.html

2. nginx配置

   - 建一个文件夹www，将HomePage这些放进去

   - 进入到 nginx 配置目录，cd /etc/nginx/conf.d/（可能nginx默认在/etc/nginx）

   - 通过 ls查看配置文件，(你之前没有配置过，下面就是空的了）

   - 然后通过 vi 命令新建一个配置文件，sudo vim yourHomePages.conf

     ```
     //正在使用的一种
     server {
         listen 1027;
         listen [::]:1027;
         root /www/static-web; # HomePages的位置，修改的地方
         server_name  _; # 如果需要改域名访问，修改server_name 为域名便可
         location / {
                 # First attempt to serve request as file, then
                 # as directory, then fall back to displaying a 404.
                 try_files $uri $uri/ =404;
         }
     }
     //第二种
     server {
         listen 80;
         root /www/static-web; # 网站根目录
         server_name  /home/www/mysite; # 要绑定的域名（或子域名）
         location / {
         }
     }
     ```

     退出之后我们需要通过命令行重启 nginx服务

     ```
     sudo nginx -s reload
     ```

     将对应端口（1027）放到防火墙外，使得外网可以访问

     

     ```
     sudo iptables -I INPUT -p tcp --dport 1027 -j ACCEPT
     sudo iptables-save
     ```

     