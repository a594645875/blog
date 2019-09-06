#### JwtFilter鉴权一直无权限
报错:无法获得header中的信息,一直无法鉴权,代码
```
final String authHeader = (String) request.getAttribute("Authorization");
if (authHeader!=null&&authHeader.startsWith("Bearer ")){
final String token = authHeader.substring(7);
Claims claims = jwtUtil.parseJWT(token);
if (claims!=null && "user".equals(claims.get("roles"))){
request.setAttribute("user_claims",claims);
```

分析:问题在于获得header信息是用`request.getAttribute("Authorization")`错误了

办法:把
```
request.getAttribute("Authorization")
```
修改成
```
request.getHeader("Authorization")
```
#### 令牌token鉴权时无法获得Authorization
描述: 无法获得头信息`Authorization`中的数据

分析: 在头信息获取的方法中写成了`String authHeader = (String) request.getAttribute("Authorization")`,获得头信息的方法为`getHeader()`,获取方法写错

解决: 把获取方法`getAttribute("Authorization")`改为`getHeader("Authorization")`
#### jjwt使用工具类时报错
报错: `base64-encoded secret key cannot be null or empty.`
分析: 秘钥为空
解决方法: 在要使用jjwt的JwtUtil工具类的模块的application.yml中添加以下信息
```
jwt:
    config:
        key: itcast
        ttl: 360000 #过期时间,不是必要
```

