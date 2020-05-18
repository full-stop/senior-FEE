# JsonP

## Introduction

通过动态插入 `script` 标签并利用其不受浏览器同源策略的限制以 `Get` 方式来请求资源。

## Install

``` text
npm install jsonp
```

## Used

``` JS
// JavaScript
import JsonP from 'jsonp';

JsonP(url: String, opts: Object, Fn: Function);
```

## Example

``` js
JsonP('http://api.map.baidu.com/telematics/v3/weather?location=上海&output=JSON&ak=fDYOMrP456lfGsUlN29lDRxTkszyrQjE&callback=test', {
    param: 'v=1',
    name: 'JsonPCallback'
}, function(err, data) {
    //can Promise Return
})
```

