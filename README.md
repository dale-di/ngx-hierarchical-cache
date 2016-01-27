#Nginx分级缓存的配置方法

主要目的是实现不同的缓存级别。设备可以当做缓存使用的存储空间有：内存，ssd，机械硬盘。
##缓存过程：
1. 当第一次请求一个可缓存url（Aurl）时，会被缓存到机械硬盘。可以多个硬盘，通过proxy_path目录下的配置文件配置多个cache zone。然后再通过upstream的一致性哈希调度cache zone。
2. Aurl在15分钟内，被请求了3次（proxy_cache_min_uses 3），就会被缓存到ssd。当然，机械盘上的缓存也继续存在。当Aurl在15分钟内没有任何请求时，缓存就会被清除。
3. Aurl在15分钟内，被请求了5次（srcache_min_uses 5），就会被缓存到redis。缓存时间由$exptime控制。

##配置说明：
1. 针对srcache-nginx-module模块做了一些简单修改。增加了srcache_min_uses配置。
2. 针对缓存内容的大小设置：client_max_body_size和srcache_store_max_size。避免在srcache_store时，出现413错误。
3. 防盗链只是提供了一种参考。可以根据自己的需求修改。
4. 分别针对不同的缓存目标添加了缓存刷新接口：<br>
/purge/(mem|ssd|hd)(/.*) <br>
\# /purge/mem/ 刷新redis缓存。<br>
\# /purge/ssd/ 刷新ssd缓存。<br>
\# /purge/hd/ 刷新机械硬盘缓存。<br>
例如： /purge/ssd/vod/.../abc.ts<br>
