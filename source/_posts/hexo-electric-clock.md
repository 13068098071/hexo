---
title: hexo-electric-clock
date: 2021-12-23
description: hexo-electric-clock 电子时钟插件配置
cover: https://raw.githubusercontent.com/13068098071/picode/main/img/8.jpg
tags: butterfly
categories: hexo
---
## hexo-electric-clock 电子时钟插件

### 效果图

![时钟](https://raw.githubusercontent.com/13068098071/picode/main/img/image-20211223195858557.png)

### 插件安装

```shell
npm i hexo-electric-clock --save
```

### 在主_config配置

```yaml
electric_clock:
  priority: 5
  enable: true
  enable_page: all
  layout:
    type: class
    name: sticky_layout
    index: 0
  temple_html: '<div class="card-widget card-clock"><div class="card-glass"><div class="card-background"><div class="card-content"><div id="hexo_electric_clock"><img id="card-clock-loading" src="https://cdn.jsdelivr.net/gh/Zfour/Butterfly-clock/clock/images/weather/loading.gif" style="height: 120px; width: 100%;" data-ll-status="loading" class="entered loading"></div></div></div></div></div>'
```

### 参数说明

>enbale

```shell
# 是否开启插件
enable: true
```

> enable_page

```shell
# 路由地址，all 代表全局开启。如 / 代表主页。
enable_page: all
```

> priority

```shell
# 插件的叠放顺序，数字越大，叠放约靠前。
priority: 5 # 这里是参数
```

> layout

```shell
# 参数：type; （class&id）
# 参数：name;
# 参数：index；（数字）
# 含义：如果说 electric_clock 是一幅画，那么这个 layout 就是指定了哪面墙来挂画
# 而在 HTML 的是世界里有两种墙分别 type 为 id 和 class。
# 其中在定义 class 的时候会出现多个 class 的情况，这时就需要使用 index，确定是哪一个。
# 最后墙的名字即是 name;

layout:
    type: class
    name: sticky_layout
    index: 0
```

> temple_html

```html
<div class="card-widget card-clock">
  <!-- 挂载容器 -->
  <div class="card-glass">
    <div class="card-background">
      <div class="card-content">
        <div id="hexo_electric_clock">
          <img
            id="card-clock-loading"
            src="https://cdn.jsdelivr.net/gh/Zfour/Butterfly-clock/clock/images/weather/loading.gif"
            style="height: 120px; width: 100%;"
            data-ll-status="loading"
            class="entered loading"
          />
        </div>
      </div>
    </div>
  </div>
</div>
```

