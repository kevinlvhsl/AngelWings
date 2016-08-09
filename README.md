# AngelWings
这是一个简单的个人兴趣网站，初衷是希望与所有的篮球爱好者们一起交流篮球兴趣，观点。一起评论NBA，精彩比赛。
希望各位爱好篮球及爱好运动的朋友多多提出宝贵意见，多多发表帖子和观点。







 plan  ing
  新闻列表准备抓取腾讯新闻的用nodejs抓取
   原文 [http://blog.csdn.net/zhangxin09/article/details/50663592](http://blog.csdn.net/zhangxin09/article/details/50663592)
> 互联网 Web 就是一个巨大无比的数据库，但是这个数据库没有一个像 SQL 语言可以直接获取里面的数据，因为更多时候 Web 是供肉眼阅读和操作的。如果要让机器在 Web 取得数据，那往往就是我们所说的“爬虫”了。现在项目需要，所以研究研究，把大概的过程和“坑”记录下来，也欢迎大渣批评和补充。

> 爬虫的思路十分简单：

按照一定的规律发送 HTTP 请求获得页面 HTML 源码（必要时需要加上一定的 HTTP 头信息，比如 cookie 或 referer 之类）
利用正则匹配或第三方模块解析 HTML 代码，提取有效数据
将数据持久化到数据库中。
上述过程虽描述得简单，但在实际过程遇到的问题还是不少的，世界上不存在完美的、一致的、通用的抓取工具。为了不同的目的，需要定制不同的代码。下面我们逐一分析。

解析 HTML

最基本的发送 HTTP 请求获得 HTML 代码，使用 node 自带的 http.request 功能即可。如果是爬简单的内容，比如获得某个指定 id 元素中的内容（常见于抓去商品价格），那么正则足以完成任务。但是对于复杂的页面，尤其是数据项较多的页面，使用 DOM 会更加方便高效。

而 node.js 最好的 DOM 实现非 cheerio [2]莫属了。其实 cheerio 应该算是 jQuery 的一个针对 DOM 操作优化和精简的子集，包含了 DOM 操作的大部分内容，去除了其它不必要的内容。使用 cheerio 你就可以像用普通 jQuery 选择器那样选择你需要的内容。

cheerio 使用方法如下所示，首先引入 cheerio = require('cheerio')。


> jsdom 也可以。

var jsdom = require("jsdom");

jsdom.env({
  url: "http://v.youku.com/v_show/id_XMTQ3Mzc1MTczNg==.html?from=y1.6-85.3.1.14b31ff68f3311e5a080&x",
//  ['http://static.youku.com/h5/player/embed/unifull/unifull_.js'],
//  {
    userAgent:'Mozilla/5.0 (iPhone; CPU iPhone OS 6_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/6.0 Mobile/10A5376e Safari/8536.25'
//  },
  ,done:function (err, window) {
    console.log(err);
    setTimeout(function(){
      
      console.log("there have been", window.document.body.innerHTML);
    }, 2000);
  }
});

> 使用 request.js

request.js 是对 http.request 更高级的封装。如果结合 cheerio 使用，是这样子的。

// 打开详情页
var request = require('request'), cheerio = require('cheerio');

function loadInfo(url){
  request({
    url: url,
    headers: {
      'User-Agent': userAgent
    }
  }, function fetch(error, response, body) {
    if (!error && response.statusCode == 200) {
      var $ = cheerio.load(body, {decodeEntities: false});
      
      var video = $('video');
      console.log(video);
    } else {
      console.log('解析 HTML 错误或通讯故障。');
    }
  });
}
