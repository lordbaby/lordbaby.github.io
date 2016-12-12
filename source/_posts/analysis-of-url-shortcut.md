title: 解析url的快捷方式
date: 2016-05-18 14:41:37
tags: 
categories: JavaScript
---

解析url的快捷方式，这个简短的函数返回一个包含你想从URL检索所有可能的信息的对象。依赖DOM

<!--more-->

```javascript

function parseURL(url) {
    var a =  document.createElement('a');
    a.href = url;
    return {
        source: url,
        protocol: a.protocol.replace(':',''),
        host: a.hostname,
        port: a.port,
        query: a.search,
        params: (function(){
            var ret = {},
                seg = a.search.replace(/^\?/,'').split('&'),
                len = seg.length, i = 0, s;
            for (;i<len;i++) {
                if (!seg[i]) { continue; }
                s = seg[i].split('=');
                ret[s[0]] = s[1];
            }
            return ret;
        })(),
        file: (a.pathname.match(/\/([^/?#]+)$/i) || [,''])[1],
        hash: a.hash.replace('#',''),
        path: a.pathname.replace(/^([^\/])/,'/$1'),
        relative: (a.href.match(/tps?:\/\/[^\/]+(.+)/) || [,''])[1],
        segments: a.pathname.replace(/^\//,'').split('/')
    };
}


    //举例
    //
    var myURL = parseURL('http://abc.com:8080/dir/index.html?id=255&m=hello#top');
     
    console.log(myURL.file);     // = 'index.html'
    console.log(myURL.hash);     // = 'top'
    console.log(myURL.host);     // = 'abc.com'
    console.log(myURL.query);    // = '?id=255&m=hello'
    console.log(myURL.params);   // = Object = { id: 255, m: hello }
    console.log(myURL.path);     // = '/dir/index.html'
    console.log(myURL.segments); // = Array = ['dir', 'index.html']
    console.log(myURL.port);     // = '8080'
    console.log(myURL.protocol); // = 'http'
    console.log(myURL.source);   // = 'http://abc.com:8080/dir/index.html?id=255&m=hello#top
```

兼容IE6。

