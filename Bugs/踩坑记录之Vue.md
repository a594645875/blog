#### wevpack打包时出错
- 问题:webpack无法打包,报一下错误
```
Invalid configuration object. Webpack has been initialised using a configuration object that does not match the API schema.
 - configuration.output.path: The provided value "./js/bundle.js" is not an absolute path!
   -> The output directory as **absolute path** (required).
```
- 分析:提示说需要用绝对路径,但是开发项目一般不能用绝对路径,
- 解决: 在Stack Overflow上找到了修改打包配置文件`webpack.config.js`中的路径,用`__name`代替项目地址
```
output: {    
    path: __dirname + "/dist/js",    
    filename: "bundle.js"
},
```
然后就打包成功了.


