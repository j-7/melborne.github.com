---
layout: post
title: "Macのターミナルで顔が変わると..."
description: ""
category: 
tags: 
date: 2014-04-29
published: true
---
{% include JB/setup %}

かわいい。

    ruby -e 'trap(:INT){print"\e[?25h\e[0;0H";exit(0)};x,y=[`tput lines`,`tput cols`].map{|n|n.to_i/2};print"\e[?25l";(0x1F600..0x1F640).cycle{|w|print"\e[2J\e[#{x};#{y}H",w.chr("utf-8");sleep(0.3)}'


![emoji noshadow](http://gifzo.net/7N6JmQxbDL.gif)

> [7N6JmQxbDL.gif - Gifzo](http://gifzo.net/7N6JmQxbDL "7N6JmQxbDL.gif - Gifzo")

---

関連記事：

[「単語が目に飛び込んできてすごい速度で文章を..」ってやつをRubyでやると...]({{ BASE_PATH }}/2014/03/01/spritz-with-ruby-oneliner/ "「単語が目に飛び込んできてすごい速度で文章を..」ってやつをRubyでやると...")

[「単語が目に飛び込んできてすごい速度で文章を..」ってやつをRubyで..を改良すると...]({{ BASE_PATH }}/2014/03/01/spritz-with-ruby-oneliner-2/ "「単語が目に飛び込んできてすごい速度で文章を..」ってやつをRubyで..を改良すると...")

---

 [単語が目に飛び込んできてすごい速度で文章を読めるようになる「Spritz」 - GIGAZINE](http://gigazine.net/news/20140228-spritz/ "単語が目に飛び込んできてすごい速度で文章を読めるようになる「Spritz」 - GIGAZINE")

---

(追記: 2014-4-30) 

翌日に書いた「[Macのターミナルで顔が降る]({{ BASE_PATH }}/2014/04/30/let-it-smile-in-the-terminal/ "Macのターミナルで顔が降る")」の評判が異常に良いので、便乗できないかとタイトルを**改ざん**しました:-)
