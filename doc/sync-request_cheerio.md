```
'use strict';

var cheerio = require('cheerio')
var request = require('sync-request');

var html = '';
html = request('GET', page_url).getBody().toString();
var $ = cheerio.load(html);
$(".js-yxk-yxmc a").each(function(index, element){
  // xxxx
})
```
