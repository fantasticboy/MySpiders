#爬取目标：

[廖雪峰的python3的教学课程](https://www.liaoxuefeng.com/wiki/1016959663602400)

# 爬取工具：

使用scrapy爬取

#策略：

根据目标网站的布局，只爬取教程的主体内容，评论等舍去，因为教程有多页，每页都有一个标题.

方案1.可以通过提取下一页的url来实现翻转到下一页的目标，然后提取目标的主体内容。

方案2.通过提取旁栏里的目录里的链接，再逐个链接去提取目标页的主体内容。

最后采取方案2，将提取到的内容，不做内容筛选和HTML标签删除，直接通过拼接HTML的架构标签，使用HTML格式存储成网页格式。

# 问题和解决方法：

在使用方案1爬取时，碰到该网站的下一页，没有被加载出来，使用xpath匹配目标HTML标签时，获取的内容为空。

了解到scrapy加载的网页内容与右键网站-->网页源代码，相同。

解决方法：使用selenium来模拟浏览器，等加载完成之后，再提取"下一页"的URL

# 源码

crawlPyTutorial3.py

```python
# -*- coding: utf-8 -*-
import scrapy
from bs4 import BeautifulSoup
from crawlLiaoXuefengPy3.items import Crawlliaoxuefengpy3Item
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from lxml import etree

class Crawlpytutorial3Spider(scrapy.Spider):
    name = 'crawlPyTutorial3'
    # allowed_domains = ['liaoxuefeng.com']
    start_urls = ['https://www.liaoxuefeng.com/wiki/1016959663602400']


    def parse(self, response):
        preUrl = 'https://www.liaoxuefeng.com'
        urlDatas = response.xpath("//ul[@id='x-wiki-index']//a/@href").extract()
        for urlData in urlDatas:
            # print(type(urlData))
            yield scrapy.Request(url=preUrl+urlData,callback=self.secondParse)

    def secondParse(self,response):
        item = Crawlliaoxuefengpy3Item()
        # print(response.text)
        # 标题
        title = response.xpath('//*[@id="x-content"]/h4/text()').extract()[0]
        item['title'] = title
        #内容
        soup = BeautifulSoup(response.text)
        prehtml = "<html><head><title>"+title+"</title></head><body>"
        posthtml = "</body></html>"
        item['content'] = prehtml+soup.select('div #x-content')[0].prettify()+posthtml
        # print(type(soup.select('div #x-content')[0].prettify()))

        yield item

        #通过selenium Chrome模拟，加载出整个页面，再筛选出下一页的url
        # browser = webdriver.Chrome("/Users/dongheng/Downloads/chromedriver")#加载Chrome驱动
        # browser.get(response.url)                                           #获得目标网页,这时为str类型
        # page = etree.HTML(browser.page_source)                              #转换成HTML格式
        # url = page.xpath("//div[@class='x-wiki-prev-next uk-clearfix']/a/@href")[0]  #使用xpath提取
        # print(url)

        # print(response.text)
```

Settings.py

```python
# -*- coding: utf-8 -*-

# Scrapy settings for crawlLiaoXuefengPy3 project
#
# For simplicity, this file contains only settings considered important or
# commonly used. You can find more settings consulting the documentation:
#
#     https://doc.scrapy.org/en/latest/topics/settings.html
#     https://doc.scrapy.org/en/latest/topics/downloader-middleware.html
#     https://doc.scrapy.org/en/latest/topics/spider-middleware.html

BOT_NAME = 'crawlLiaoXuefengPy3'

SPIDER_MODULES = ['crawlLiaoXuefengPy3.spiders']
NEWSPIDER_MODULE = 'crawlLiaoXuefengPy3.spiders'


# Crawl responsibly by identifying yourself (and your website) on the user-agent
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.77 Safari/537.36'

# Obey robots.txt rules
ROBOTSTXT_OBEY = True

# Configure maximum concurrent requests performed by Scrapy (default: 16)
#CONCURRENT_REQUESTS = 32

# Configure a delay for requests for the same website (default: 0)
# See https://doc.scrapy.org/en/latest/topics/settings.html#download-delay
# See also autothrottle settings and docs
DOWNLOAD_DELAY = 1
# The download delay setting will honor only one of:
#CONCURRENT_REQUESTS_PER_DOMAIN = 16
#CONCURRENT_REQUESTS_PER_IP = 16

# Disable cookies (enabled by default)
#COOKIES_ENABLED = False

# Disable Telnet Console (enabled by default)
#TELNETCONSOLE_ENABLED = False

# Override the default request headers:
#DEFAULT_REQUEST_HEADERS = {
#   'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
#   'Accept-Language': 'en',
#}

# Enable or disable spider middlewares
# See https://doc.scrapy.org/en/latest/topics/spider-middleware.html
#SPIDER_MIDDLEWARES = {
#    'crawlLiaoXuefengPy3.middlewares.Crawlliaoxuefengpy3SpiderMiddleware': 543,
#}

# Enable or disable downloader middlewares
# See https://doc.scrapy.org/en/latest/topics/downloader-middleware.html
#DOWNLOADER_MIDDLEWARES = {
#    'crawlLiaoXuefengPy3.middlewares.Crawlliaoxuefengpy3DownloaderMiddleware': 543,
#}

# Enable or disable extensions
# See https://doc.scrapy.org/en/latest/topics/extensions.html
#EXTENSIONS = {
#    'scrapy.extensions.telnet.TelnetConsole': None,
#}

# Configure item pipelines
# See https://doc.scrapy.org/en/latest/topics/item-pipeline.html
ITEM_PIPELINES = {
   'crawlLiaoXuefengPy3.pipelines.Crawlliaoxuefengpy3Pipeline': 300,
}

# Enable and configure the AutoThrottle extension (disabled by default)
# See https://doc.scrapy.org/en/latest/topics/autothrottle.html
#AUTOTHROTTLE_ENABLED = True
# The initial download delay
#AUTOTHROTTLE_START_DELAY = 5
# The maximum download delay to be set in case of high latencies
#AUTOTHROTTLE_MAX_DELAY = 60
# The average number of requests Scrapy should be sending in parallel to
# each remote server
#AUTOTHROTTLE_TARGET_CONCURRENCY = 1.0
# Enable showing throttling stats for every response received:
#AUTOTHROTTLE_DEBUG = False

# Enable and configure HTTP caching (disabled by default)
# See https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#httpcache-middleware-settings
#HTTPCACHE_ENABLED = True
#HTTPCACHE_EXPIRATION_SECS = 0
#HTTPCACHE_DIR = 'httpcache'
#HTTPCACHE_IGNORE_HTTP_CODES = []
#HTTPCACHE_STORAGE = 'scrapy.extensions.httpcache.FilesystemCacheStorage'
```

Pipelines.py

```python
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://doc.scrapy.org/en/latest/topics/item-pipeline.html


class Crawlliaoxuefengpy3Pipeline(object):
    def process_item(self, item, spider):
        fp = open("/Users/username/Desktop/py3Tutorial/"+item['title']+'.html','w')
        fp.write(item['content'])
        fp.close()

        return item
```

items.py

```python
# -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# https://doc.scrapy.org/en/latest/topics/items.html

import scrapy


class Crawlliaoxuefengpy3Item(scrapy.Item):
    #题目
    title = scrapy.Field()
    #内容
    content = scrapy.Field()
```