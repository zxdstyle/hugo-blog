---
title: "Vue Cli 项目全局使用 less 变量"
date: 2020-05-13
lastmod: 2020-05-13
draft: false
tags: ["vue", "less"]
categories: ["vue"]
author: "zxdstyle"
contentCopyright: '本站所有文章无特殊说明都是原创，转载请注明出处<a rel="license noopener" href="https://www.zxdstyle.cn" target="_blank">www.zxdstyle.cn</a>'

---

# 安装插件

```bash
yarn add less less-loader style-resources-loader vue-cli-plugin-style-resources-loader -D
```

# 新增 less 变量文件

```less
// file variable.less
@color: #fff;
```


# 调整 vue.config.js 文件

```js
const path = require("path")

module.exports = {
	pluginOptions: {
		"style-resource-loader":{
			preProcessor: "less",
			patterns: [path.resolve(__dirname, "./src/assets/less/variable.less")]
		}
	}
}

```

# 使用

```vue
<template>
	<div class="test"></div>	
</template>

<script></script>

<style lang="less" scoped>
.test {
	background: @color;
}
</style>

```