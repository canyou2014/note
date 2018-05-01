# 开源分布式爬虫项目`gocrawler`代码阅读笔记

### 设计思路

`gocrawler`是一个遵守`robots.txt`规则的爬取Web页面的工具。

*   首先读取`robots.txt`数据，然后按顺序在同一时刻进行一次请求，每次请求之间间隔指定的延迟时间。每个host之间没有约束，所以多个worker能够独立进行。
*   visitor 函数在worker的goroutine被调用，为什么这么做是可行的，是因为延迟时间远远大于解析内容的时间。
*   没有延迟时间也是支持的，但是不理想
*   Extender 接口主要提供扩展，增强爬虫的功能

### 模块

#### crawler

该模块是整个项目的核心模块，控制着所有的执行功能，包括产生worker协程和管理URL队列。

*   **NewCrawler(Extender)** : 根据指定的Extender实例，创建一个Crawler实例
*   **NewCrawlerWithOptions(Options)**: 根据预先创建的Options实例，创建一个Crawler实例

两种方式会终止爬取：

*   没有URL时
*   达到了`Options.Maxvisit`设置的最大访问时

主要通过如下类型作为参数传入：

*   string
*   []string
*   *url.URL 
*   []*url.URL
*   map[string]interface{}
*   map[*url.URL]interface{}

gocrawl.S 和 gocrawl.U分别表示string的map和URLs的map

#### Options

包含一些基本配置和指定的Extender

#### Hooks 和 customizations

Options提供hoos和customizations。其中UserAgent和RobotUserAgent需要设置成适合自己的配置

*   UserAgent： 读取网页的user-agent。默认设置为Windows上的Firefox 15的user-agent。需要更改为包含自己的robot名字和关联的链接。
*   RobotUserAgent：robot的user-agent字符串， 用来在robots.txt中找到相匹配的规则。
*   MaxVisit： 读取的网页的最大数量。通过在达到最大访问数量时，发送一个停止信号来终止任务，但是因为停止时，可能还有一些任务正在运行，要等到所有任务跑完为止才会停止，所以实际的最大反问数可能比设置的最大访问数要大（MaxVisits + number of active workers）
*   EnqueueChanBuffer: URLs 队列的大小，默认是100
*   HostBufferFactor： 任务和通道的大小因子，当将SameHostOnly设置为false时才生效（当SameHostOnly设置为true时，Crawler明确知道基于种子URLs的host数量，但是为false时，这个数量可能会程指数增长）
*   CrawlerDelay： 两个host访问请求之间的等待时间。该时间从接受到来自host的响应开始计算。默认使用robots.txt中设置的值。该值也能通过ComputerDelay扩展来设置
*   WorkerIdleTTL： 任务被清除之前的最大存活时间。延迟时间不算在这个时间内，该时间指的是任务可用，但没有URLs可操作的时间。
*   SameHostOnly：限制队列中指向同一个host的URLs。默认为true
*   HeadBeforeGet： 告诉Crawler发出HEAD请求，默认为false
*   URLNormalizationFlags： 格式化URL的flag，使用`purell`包，默认设置为`purell.FlagsAllGreedy`
*   LogFlags: 打印日志的登记，默认只打印错误日志
*   Extender: 扩展。必须被指定，提供一个默认扩展DefaultExtender

#### Extender


