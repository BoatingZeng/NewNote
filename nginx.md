## 简单配置示例
```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    
    default_type  application/octet-stream;

    proxy_cache_path  cache/cache2  levels=1:2 keys_zone=cache2:100m inactive=10m max_size=1g;

    server {
        listen       80;
        server_name  localhost;
      
        add_header X-Cache $upstream_cache_status;

        location / {
            proxy_pass http://localhost:18189;
        }

        location /home {
            proxy_pass http://localhost:18189;

            proxy_cache cache2;

            proxy_cache_key $host$uri$is_args$args;

            proxy_cache_valid 200 1m;
        }
        
        location /category {
            proxy_pass http://localhost:18189;

            proxy_cache cache2;

            proxy_cache_key $host$uri$is_args$args;

            proxy_cache_valid 200 1m;
        }
        
        location /assets/ {
            root public;
        }
    }
}
```

## proxy_cache_path的说明：
```
proxy_cache_path  cache/cache2  levels=1:2 keys_zone=cache2:100m inactive=10m max_size=1g;
```
1. `cache/cache2`：缓存目录
2. `levels=1:2`：缓存目录结构
3. `inactive=10`：10分钟内没被访问的缓存会被删除
4. `max_size=1g`：缓存最大1g
5. `keys_zone=cache2:100m`：`cache2`这个名字下面会用到，这个名字是自己起的。`100m`的说明参考文档。


## location /home块的说明：

和`location /category`一样，它们会匹配以`/home`和`/category`开头的url，比如：`/category/sub?ccode=story`和`/category/ajax?ccode=story`

```
proxy_cache cache2;
```
表示使用cache2这个缓存

```
proxy_cache_key $host$uri$is_args$args;
```
请参考文档。带`$`前缀的是Nginx变量，上面这个`$host$uri$is_args$args`会变成类似`localhost:18189/home?query=test`或`localhost:18189/category`。

**注意：** 下面几个URL会命中不同缓存。

1. `localhost:18189/home?query=test`
2. `localhost:18189/home?query=test2`
3. `localhost:18189/home?query=test`
4. `localhost:18189/home`

```
proxy_cache_valid 200 1m;
```
请参考文档。这样设置的话，第一次请求`/home`，会走node，1分钟内再请求不走node，1分钟后会再走node。

## location /assets/块的说明：

这是静态文件的配置。

把项目的public移到了Nginx目录下，也就是原项目没有public目录了。这样，访问`localhost:18189`是获取不到静态文件的，页面显示不正常，`18189`是node项目的端口。访问`80`，即Nginx，是可以获取静态文件，页面正常显示。

详细可参考：

http://freeloda.blog.51cto.com/2033581/1288553

https://www.nginx.com/blog/nginx-caching-guide/

http://nginx.org/en/docs/

## 详细配置示例

```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;

    default_type  application/octet-stream;

    proxy_cache_path  cache/cache_dm  levels=1:2 keys_zone=cache_dm:100m inactive=1m max_size=1g;

    server {
        listen       80;
        server_name  localhost;
      
        add_header X-Cache $upstream_cache_status;
        
        location /assets {
            root F:/work/com/dongshi/shanghai/iptv-4k-dongman/public;
        }

        location / {
            proxy_pass http://localhost:18189;
        }

        location /home {
            proxy_pass http://localhost:18189;

            proxy_cache cache_dm;

            proxy_cache_key $host$uri$is_args$args;

            proxy_cache_valid 200 1m;
        }

        location /category {
            proxy_pass http://localhost:18189;

            proxy_cache cache_dm;

            proxy_cache_key $host$uri$is_args$args;

            proxy_cache_valid 200 1m;
        }

        location /search {
            proxy_pass http://localhost:18189;

            proxy_cache cache_dm;

            proxy_cache_key $host$uri$is_args$args;

            proxy_cache_valid 200 1m;
        }

        location /search/do_search {
            proxy_pass http://localhost:18189;
        }

        location /product {
            proxy_pass http://localhost:18189;

            proxy_cache cache_dm;

            proxy_cache_key $host$uri$is_args$args;

            proxy_cache_valid 200 1m;
        }

        location /product/userInfo {
            proxy_pass http://localhost:18189;
        }
    }
}
```

## 需要调整的参数

1. `cache/cache_dm`：缓存文件路径
2. `keys_zone=cache_dm:100m`：缓存区名字和Key大小，100m应该够了
3. `inactive=1m`：不活跃缓存被删除的时间
4. `max_size=1g`：缓存最大大小
5. `proxy_cache cache_dm`：这个参数填缓存区名字
6. `proxy_cache_valid 200 1m`：200是响应码，只对200的设置应该就够了，1m是缓存1分钟内可用
7. `proxy_pass http://localhost:18189`
8. `add_header X-Cache $upstream_cache_status`：用来在响应头里标识有没有命中缓存的，不用更改
9. 静态文件的配置

## 关于inactive和proxy_cache_valid

inactive的时间表示一个文件在指定时间内没有被访问过，就从存储系统中移除，不管你proxy_cache_valid里设置的时间是多少。而proxy_cache_valid在保证inactive时间内被访问过的前提下，最长的可用时间。proxy_cache_valid定义的其实是一个绝对过期时间(第一次缓存的时间+配置的缓存时间)，到了这个点，对象就被认为是过期，然后去后端重取数据，尽管它被访问的很频繁(即所谓的inactive时间内)。expires呢，它不在这个过期控制体系内，它用在发给客户端的响应中，添加"Expires"头。

## 关于缓存的储存

缓存用文件的形式储存在磁盘内。
