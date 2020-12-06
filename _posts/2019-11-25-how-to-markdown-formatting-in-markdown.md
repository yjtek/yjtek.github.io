---
layout: post
title: How to Markdown - Formatting in Markdown
categories: []
tags: []
description: An overview for markdown syntax
published: true
fullview: false
comments: true
author: Yong Jian
---

### Markdown Syntax: A quick overview
<br>

This is the first post, just to get familiar with Markdown syntax and options. I've thrown together a quick reference table for text formating in markdown. Will add on to it over time if needed, but for now this is adapted from the [cheatsheet](https://rstudio.com/wp-content/uploads/2016/03/rmarkdown-cheatsheet-2.0.pdf) put together by RStudio.
<br><br>

{:class="table table-bordered"}
|Description|Code|Output|
|-----|-----|-----|
|Plain text|Plain text|Plain text|
|Use two spaces at the end of line to signal new paragraph|New &nbsp;&nbsp; paragraph|New <br> paragraph|
|Italics|\*Italics\*|*Italics*|
|Bold|\*\*Bold\*\*|**Bold**|
|Quoting verbatim, or package name|&#96;dplyr&#96;|`dplyr`|
|Superscript/Subscript|Normal &#60;sup&#62;superscript&#60;/sup&#62; &#60;sub&#62;subscript&#60;/sub&#62;;|Normal <sup>superscript</sup> <sub>subscript</sub>|
|Strikethrough|&#126;&#126;strikethrough&#126;&#126;|~~strikethrough~~|
|Use forward slash to escape characters|&#92;* &#92;_ &#92;&#92;|\* \_ \\|
|Endash or emdash|Endash: &#45;&#45; Emdash: &#45;&#45;&#45;|Endash: --; Emdash: ---|
|Use \$ to write latex equations| \\$\\$E = mc^{2}\\$\\$ |$$E = mc^{2}$$|
|Block quote|&#62; Block quote| > Block quote (can't block quote in table) |
|Use \# for headers. 6 sizes available|\#\#\# Header|### Header|
|You can also use an \#anchor to link back to a specific header in the page|\#\#\# Header \{\#anchor\}|### Header|
|Create a link back to anchor|Jump to &#91;Header&#93;&#40;\#anchor&#41;|Jump to [Header](#anchor)|
|Add hyperlink|&#91;link&#93;&#40;google.com&#41;|[link](google.com)|
|Inline comments are possible|\<\!\-\-Text comment\-\-\> blah|<!--Text comment--> blah|
|You can insert pics from the web... |\!&#91;Caption&#93;&#40;link&#41;|![Caption](https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png)|
|...or from a local directory |\!&#91;Caption&#93;&#40;path&#41;|![Caption](/img/2019-11-25-testlogo.png)|
|Insert order/unordered lists|\* unordered list <br> 1. Ordered list|*  unordered list <br> 1. Ordered list|
|Footnote|A footnote [&#94;Footnote here] <br> \[^Footnote]: Footnoted|A footnote[^Footnote]|


[^Footnote]: Footnoted
