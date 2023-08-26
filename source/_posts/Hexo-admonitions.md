---
title: Hexo admonitions
date: 2023-08-26 22:27:26
tags:
---

For a blog post that I'm writing I wanted to use admonitions. Because setting up [hexo](https://hexo.io/) to use admonitions requires a tiny dose of manual tweaking I'm going to keep a note about it here in case I have to do it again in the future.

The steps to have admonitions are the following. First install a hexo plugin:

```shell
npm install hexo-admonitions --save
```

The next and final steps is to copy the [required css](https://github.com/lxl80/hexo-admonition#%E6%A0%B7%E5%BC%8F%E9%85%8D%E7%BD%AE) styles to the current theme. I created a new stylus file called `_admonitition.styl` and imported it from the main stylus file `style.styl`:

```stylus
@import "_admonitition"
```
