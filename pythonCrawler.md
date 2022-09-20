# 一、网站背景调研

爬取网站前需要先对目标站点的规模和结构进行调研。主要分为`robots.txt`和`Sitemap`文件。



* **robots.txt**

`robots.txt`文件描述了网站爬取的限制，可以将爬虫被封禁的可能性降至最低，也可以了解网站结构。



* **Sitemap**

`Sitemap`文件可以帮助爬虫定位网站最新内容，而无需爬取所有网页（[网站地图标准定义](https://sitemaps.org/protocol.html)）。“网站地图”提供了所有网页链接，但可能存在缺失、过期、不完整的问题。



* **估算网站大小**

网站大小会影响我们爬取方式的选择。一般爬取只有几百个URL的网站，效率没有那么重要；但如果某网站拥有数百万个URL，使用串行下载可能需要数月时间，这时就需要用到分布式下载方式。

估算网站大小可以通过检查`Google爬虫`的结果，因为Google可能已经怕去过我们感兴趣的网站，并做过相关工作了。



* **识别网站所用技术**

网站所用到的技术类型也会影响到我们爬取方式的选择。

检查网站构建技术类型的模块是`detectem`，该模块需要python3.5以上版本以及`Docker`。

安装（[官网安装](https://www.docker.com/)）完Docker后可以利用下列命令拉取最新Docker镜像：

```shell
docker pull scrapinghub/splash
pip install detectem
```

`detectem`模块基于`splash`，使用一系列请求和响应来探测网站所用技术。

`splash`是由`scrapinghub`开发的一个脚本化浏览器。

想要运行该模块只需使用`det`命令即可。例如：

```python
det www.baidu.com
```



* **网站所有者**

有的所有者可能会封禁爬虫，最好将下载速度控制得保守些。

可以利用`WHOIS`协议查询域名的注册者是谁。python中有一个对应该协议的封装库`python-whois`，使用前用pip安装即可（`pip install python-whois`）。

```python
import whois
print(whois.whois('www.baidu.com')) #这里用百度域名来测试
```

结果如下：

<img src="https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202209201741601.png" alt="image-20220920174155514"  />

可以看到最后的`"org"`字段是：北京百度公司。



* **三种网站爬取方法**
    1. 爬取`sitemap`网站地图
    2. 使用数据库ID遍历每个网页
    3. 跟踪网页链接

