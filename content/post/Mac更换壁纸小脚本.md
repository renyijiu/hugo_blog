---
title: "Mac更换壁纸小脚本"
date: 2018-03-19T21:56:17+08:00
draft: false
keywords: []
description: "ruby小脚本，更换电脑壁纸"
tags: []
categories: ['ruby']
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
mathjax: false
---

最近偶然逛到[unsplash](https://unsplash.com/)这个免费的图片网站，发现图片质量不错，另外还友好的提供API接口，就想着顺带写一个小脚本去自动更换电脑壁纸（换一张壁纸，换一个心情😊）。

<!--more-->

大致过程如下：

1. 通过unsplash接口获取一张图片下载到本地
2. 通过AppleScript设定图片作为桌面壁纸
3. 通过crontab设定定时任务，定时执行脚本去更换壁纸

过程很简单，所以代码很简单（代码只是自己电脑上跑，所以很多东西如错误处理未考虑周全）

```ruby
require 'net/http'
require 'json'

class Wallpaper
  
  def initialize
    @default_url = "https://api.unsplash.com/photos/random"
    @client_id = "SECRET_KEY" # unsplash网站申请的key
    @collection = "909298" # 图片类别，这里默认是mac壁纸

    @default_filepath = "<path>/wallpaper.jpg" # 默认图片保存地址
  end
  
  def main
    p "开始下载图片。。。"
    download

    p "下载完成，设置背景图片"
    set_wallpaper

    p "设置完成，去看看吧😊"
  end

  # 图片下载过程
  # 获取json，提取图片地址，下载图片
  def download
    img_url = get_image_url
    download_img(img_url) if img_url
  end

  # 基于AppleScript，设置背景图片
  def set_wallpaper(default_filepath=nil)
    default_filepath ||= @default_filepath

    osascript <<-END
      set filePath to POSIX file "#{default_filepath}"
      tell application "Finder"
        set desktop picture to alias filePath
      end tell
    END
  end

  private

  def get_image_url
    params = {client_id: @client_id, collections: @collection}
    flag, res = get(@default_url, params)
    
    if flag
      res_json = JSON.parse(res)

      return res_json.dig("urls", "raw")
    end

    false
  end

  def download_img(download_url, default_filepath=nil)
    return if !download_url
    
    default_filepath ||= @default_filepath
    File.open(default_filepath, 'w') do |f|
      flag, res = get(download_url)

      f.write res if flag
    end
  end

  def get(url, params={})
    uri = URI(url)
    uri.query = URI.encode_www_form(params)

    res = Net::HTTP.get_response(uri)
    if res.is_a?(Net::HTTPSuccess)
      [true, res.body]
    else
      [false, '请求出错了！']
    end
  end

  # 系统执行AppleScript
  def osascript(script)
    system 'osascript', *script.split(/\n/).map { |line| ['-e', line] }.flatten
  end
end

wallpaper = Wallpaper.new
wallpaper.main
```

上面就是脚本的所有代码，然后通过crontab设定定时任务执行即可达到自动更新电脑壁纸的效果。

注意的是，代码测试时发现当图片较大时，脚本执行完壁纸不会立即刷新，这个栽在坑里很久才发现问题（起初一直以为是代码问题导致未更新），需要等一会后会更新。不过相对来说，这个不影响实际效果。

另外由于使用AppleScript去执行设定壁纸的任务（个人认为这是一个比较优雅的方式，不用去使用一些看着较为侵入性的命令），所以也就是只有mac能使用，只要替换这一步其实就可以换成其他系统设定的方式即可。

最后，其实mac系统自带了2、3两步的实现，壁纸的设置中可以选定特定的文件夹，然后设置循环时间即可做到自动更新。当然图片还是需要自己提供的，所以只需要下载就够了😄。