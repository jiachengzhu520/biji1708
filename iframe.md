## vue-quill-editor富文本如何替换iframe为video

```js
this.assemblyForm.text = res.result.text//富文本内容
let iframeReg = new RegExp(`<iframe class="ql-video" frameborder="0" allowfullscreen="true"`,'g')//用new RegExp 正则匹配替换所有iframe
let iframeReg2 = new RegExp('</iframe>','g')//用new RegExp 正则匹配替换iframe
let str = this.assemblyForm.content
let res = str.replace(iframeReg,`<video controls style="width: 100%;"`).replace(iframeReg2,'</video>')//replace 用video标签替换iframe
this.assemblyForm.content=res//重新赋值
```

如何清理iframe url缓存

```js
 var ts = new Date().getTime()//获取当前时间戳
 iframeUrl = `${url}?_`+ts// url 拼接时间戳
 <iframe :src="iframeUrl" scrolling="yes" class="iframe_two"></iframe>


 
```

