# <center>利用Nginx搭建文件服务器</center>

### 修改nginx.conf
	server{
        listen       8000;
        server_name  localhost;

        charset utf-8;

        root /home/files/;
        location  / {
			//加上文件头，不然这些文件直接打开，而不是下载
            if ($request_filename ~* ^.*?\.(txt|doc|pdf|rar|gz|zip|docx|exe|xlsx|ppt|pptx|html)$){
                add_header Content-Disposition 'attachment;';
            }
        }

        autoindex   on;     #开启索引功能
        autoindex_exact_size off;  #关闭计算文件确切大小（单位bytes），只显示大概大小（单位kb、mb、gb）
        autoindex_localtime on;     #显示文件时间
    }