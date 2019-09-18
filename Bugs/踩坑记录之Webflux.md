### 浏览器不出现流式响应效果,只出现最终结果

```Java
    /**
     * 返回0-n个数据
     * @return
     */
    @GetMapping(value = "/3",produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> get3() {
        log.info("get3 start");
        Flux<String> result = Flux.fromStream(IntStream.range(1,5).mapToObj(i->{
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "flux data --" + i;
        }));
      
```

在学习Webflux的过程中,在浏览器请求`http://localhost:8080/3`的时候,页面没有出现流式动态显示,只出现最终数据.

经过排查发现和学习资料用的Springboot版本是2.0.0,练习用的是2.1.8.Release,更改成2.0.0之后就正常显示流式数据了.

有可能是改版之后的代码写法不一样.

解决办法: 把练习Springboot版本由2.1.8改为2.0.0