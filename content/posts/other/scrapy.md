---
title: "Scrapy"
date: 2022-04-08T13:18:21+08:00
author: ["loveyu"]
draft: false
categories: 
- other
tags: 
- scrapy
- python
---

# 一: 创建项目

## 1. 创建项目

```shell
scrapy startproject 项目名称
```

## 2. 创建spider

>url不需要加http

```shell
scrapy genspider filename "url" 
```

## 3. 运行

```shell
scrapy crawl filename
```



# 二: 代码

## 1. settings.py关闭robot

```python
ROBOTSTXT_OBEY = False
```



## 2. 在生成的文件中

```python
import scrapy

from scrapydemo1.items import Scrapydemo1Item


class BianSpider(scrapy.Spider):
    name = "bian"
    # 允许的域,只会在这个域下面获取数据
    allowed_domains = ["www.netbian.com"]
    # 起始url,运行后第一次访问的url
    start_urls = ["http://www.netbian.com/dongman"]
    # 多页时的url
    url = "http://www.netbian.com/dongman/index_"
    # 当前第几页
    page = 1

    # response页面的源代码
    def parse(self, response: scrapy.http.Response, *args, **kwargs):
      	# 解析
        for d in response.xpath('//div[@class="list"]/ul/li/a/img'):
            name = d.xpath('./@alt').extract_first()
            img = d.xpath('./@src').extract_first()
            item = Scrapydemo1Item()
            item['img_src'] = img
            item['name_alt'] = name
            # 返回给pipelines
            yield item
        # 下一页
        if self.page < 5:
            self.page += 1
            url = self.url + str(self.page) + ".htm"
            # 拼接好url后调用Request
            yield scrapy.Request(url=url, callback=self.parse)

```



## 3. Items.py

>解析后的数据结构类型

```python
class Scrapydemo1Item(scrapy.Item):
    img_src = scrapy.Field()
    name_alt = scrapy.Field()
```



## 4. pipelines.py

>spiders解析后返回的数据会到这里, 运行同时有多个管道, 每个管道可以有各自的任务
>
>需要在settings.py中声明出管道, 数字越小优先级越高
>
>```py
>ITEM_PIPELINES = {
>    'scrapydemo1.pipelines.ScrapyBianPipeline': 300,
>    'scrapydemo1.pipelines.ScrapyBianDownPipeline': 301,
>}
>```
>
>

```python
import urllib.request

from itemadapter import ItemAdapter


class Scrapydemo1Pipeline:
    def process_item(self, item, spider):
        return item

# 第一个管道
class ScrapyBianPipeline:
    # 启动前执行
    def open_spider(self, spider):
        self.fp = open("scrapydemo1/resource/bian.json", "w", encoding="utf-8")

    # 启动后执行
    def close_sipder(self, spider):
        self.fp.close()

    def process_item(self, item, spider):
        self.fp.write(str(item))
        return item

# 第二个管道
class ScrapyBianDownPipeline:
    def process_item(self, item, spider):
        url = item['img_src']
        name = item['name_alt']
        urllib.request.urlretrieve(url, "scrapydemo1/resource/img/" + name + ".png")
        return item

```



# 三: post请求

```python
class FySpider(scrapy.Spider):
    name = "fy"
    allowed_domains = ["fanyi.baidu.com"]

    def start_requests(self):
        url = "https://fanyi.baidu.com/sug"
        data = {"kw": "hello"}
        yield scrapy.FormRequest(url=url, formdata=data, callback=self.post_parse)

    def post_parse(self, response):
        content = response.text
        obj = json.loads(content)
        print(obj)
```



# 四: 中间件

>在settings.py中吧中间件注释取消
>
>执行顺序:
>
>```shell
>DownloaderMiddleware: from_crawler
>SpiderMiddleware: from_crawler
>DownloaderMiddleware: spider_opened
>SpiderMiddleware: spider_opened
>SpiderMiddleware: process_start_requests
>DownloaderMiddleware: process_request
>DownloaderMiddleware: process_response
>SpiderMiddleware: process_spider_input
>[结果]
>SpiderMiddleware: process_spider_output
>```
>
>

```python
DOWNLOADER_MIDDLEWARES = {
    'bdfy.middlewares.BdfyDownloaderMiddleware': 543,
}
```

```python
    def process_request(self, request, spider):
        user_agent = random.choice(USER_AGENTS_LIST)
        request.headers["User-Agent"] = user_agent
        request.meta["proxy"] = "ip"
        return None

    def process_response(self, request, response, spider):
        print(request.headers["User-Agent"])
        return response
```



# CrawlSpider

>连接提取器, 快速获取指定格式的链接,一般用于对页数链接的提取, 使用这个就不需要再用page拼接下一页的url

>创建文件命令

```shell
scrapy genspider -t crawl filename url
```

>代码

```python
class DsSpider(CrawlSpider):
    name = "ds"
    allowed_domains = ["www.dushu.com"]
    start_urls = ["https://www.dushu.com/book/1188_1.html"]

    # allow: 正则表达式, 获取所有符合这个正则表达式的链接, follow:从response提取的链接是偶需要跟进
    rules = (Rule(LinkExtractor(allow=r"book/1188_\d+\.html"), callback="parse_item", follow=True),)

    def parse_item(self, response: scrapy.http.Response):
        data = response.xpath('//div[@class="bookslist"]//img')
        for d in data:
            assert isinstance(d, scrapy.selector.Selector)
            name = d.xpath("./@alt").extract_first()
            img = d.xpath("./@data-original").extract_first()
            item = DswItem()
            item['name'] = name
            item['img'] = img
            yield item
```





# 问题

## 1. https ssl错误

```python
import ssl
ssl._create_default_https_context = ssl._create_unverified_context
```



## 2. 嵌套页面获取数据

>`yield scrapy.Request(url=img_url, callback=self.two_parse, meta={"name": name})`
>
>不是回调本身而是回调自定义的一个方法
>
>`meta={"name": name}`
>
>向回调的自定义方法传递数据,以map的形式传递
>
>`name = response.meta["name"]`
>
>取数据的方式
>
>`yield scrapy.Request(url=url, callback=self.parse)`
>
>获取多页数据不在parse中调用而是在自定义的方法two_parse中调用,但是`参数callback依旧是self.parse` 

```python
class MvSpider(scrapy.Spider):
    name = "mv"
    allowed_domains = ["www.ygdy8.net"]
    start_urls = ["https://www.ygdy8.net/html/gndy/china/index.html"]
    page = 1
    # url + page.html
    url = "https://www.ygdy8.net/html/gndy/china/list_4_"

    def parse(self, response: scrapy.http.Response, *args, **kwargs):
        data = response.xpath('//div[@class="co_content8"]//td[2]//a[2]')
        for d in data:
            assert isinstance(d, scrapy.selector.Selector)
            img = d.xpath('./@href').extract_first()
            name = d.xpath('./text()').extract_first()
            img_url = "https://www.ygdy8.net" + img
            yield scrapy.Request(url=img_url, callback=self.two_parse, meta={"name": name})

    def two_parse(self, response: scrapy.http.Response, *args, **kwargs):
        # 坑点: span标签无法使用不能加span
        img = response.xpath('//*[@id="Zoom"]//img/@src').extract_first()
        name = response.meta["name"]
        mv = DyttItem()
        mv["img"] = img
        mv["name"] = name
        yield mv
        if self.page < 5:
            self.page += 1
            url = self.url + str(self.page) + ".html"
            yield scrapy.Request(url=url, callback=self.parse)
```

## 3. span标签无法在xpath中使用



# 命令

```shell
== 创建项目
scrapy startproject 项目名称

== 创建默认spider文件
scrapy genspider filename url

== 创建CrawlSpider文件
scrapy genspider -t crawl filename url

== 运行
scrapy crawl filename

== 命令行工具
scrapy shell
```

