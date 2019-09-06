### 安装和使用vue-cli
- 安装 Node.js
请自行前往 http://nodejs.cn/download 官网下载安装
![](https://www.funtl.com/assets/Lusifer_20181224052651.png)
- 安装 Node.js 淘宝镜像加速器（cnpm）
```
npm install cnpm -g

# 或使用如下语句解决 npm 速度慢的问题
npm install --registry=https://registry.npm.taobao.org
```
![](https://www.funtl.com/assets/Lusifer_20181224053021.png)
- 安装 vue-cli
```
cnpm install vue-cli -g
```
- 测试是否安装成功
```
# 查看可以基于哪些模板创建 vue 应用程序，通常我们选择 webpack
vue list
```
![](https://www.funtl.com/assets/Lusifer_20181224053315.png)
- 创建一个基于 webpack 模板的 vue 应用程序
```
# 这里的 myvue 是项目名称，可以根据自己的需求起名
vue init webpack myvue
```
![](https://www.funtl.com/assets/Lusifer_20181224054035.png)
- 初始化并运行
```
cd myvue
npm install
npm run dev
```
![](https://www.funtl.com/assets/Lusifer_20181224060151.png)
- 测试
安装并运行成功后在浏览器输入：http://localhost:8080
![](https://www.funtl.com/assets/Lusifer_20181224060413.png)