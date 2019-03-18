---
title: Spring Boot 构建电商基础秒杀项目 (四) getotp 页面
date: 2019-03-17 09:00:00
updated: 2019-03-17 09:00:00
categories: [IT]
tags: [Java, Spring Boot]
---

> [SpringBoot构建电商基础秒杀项目](https://www.imooc.com/video/18390) 学习笔记

# BaseController 添加

```
public static final String CONTENT_TYPE_FORMED = "application/x-www-form-urlencoded";
```

# UserController 添加

需添加 @CrossOrigin 注解，解决跨域问题

```
    @Autowired
    private HttpServletRequest httpServletRequest;

    @RequestMapping(value = "/getotp", method = {RequestMethod.POST}, consumes = {CONTENT_TYPE_FORMED})
    @ResponseBody
    public CommonReturnType getOtp(@RequestParam(name="telphone") String telphone){
        // 生成 otp 验证码
        Random random = new Random();
        int randomInt = random.nextInt(99999);
        randomInt += 10000;
        String otpCode = String.valueOf(randomInt);

        // 将 otp 验证码同对应的用户关联 (暂时使用 httpsession 的方式绑定 otp 与手机号)
        httpServletRequest.getSession().setAttribute(telphone, otpCode);

        // 将 otp 验证码通过短信通道发送给用户 (省略，使用控制台输出代替)
        System.out.println(String.format("telphone = %s & otpCode = %s", telphone, otpCode));

        return CommonReturnType.create(null);
    }
```

# 新增 getotp 页面

```
<html>
    <head>
        <meta charset="UTF-8">
        <link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
    </head>
    
    <body>
        <div id="app">
            <el-row>
                <el-col :span="8" :offset="8">
                    <h3>获取 otp 信息</h3>
                    <el-form ref="form" :model="form" label-width="80px">
                        <el-form-item label="手机号">
                            <el-input v-model="form.telphone"></el-input>
                        </el-form-item>
                        
                        <el-form-item>
                            <el-button type="primary" @click="onSubmit">获取 otp 短信</el-button>
                        </el-form-item>
                    </el-form>
                </el-col>
            </el-row>
        </div>
    </body>
    
    <script src="https://unpkg.com/vue/dist/vue.js"></script>
    <script src="https://cdn.bootcss.com/axios/0.18.0/axios.min.js"></script>
    <script src="https://unpkg.com/element-ui/lib/index.js"></script>
    <script>
        var app = new Vue({
            el: '#app',
            data: {
                form: {
                    telphone: '',
                }
            },
            methods: {
                onSubmit(){
                
                    if(this.form.telphone == null || this.form.telphone == ''){
                        console.log('手机号不能为空');
                        return;
                    }
                
                    // https://www.cnblogs.com/yesyes/p/8432101.html
                    axios({
                        method: 'post',
                        url: 'http://localhost:8080/user/getotp', 
                        data: this.form, 
                        params: this.form,
                        headers: {'Content-Type': 'application/x-www-form-urlencoded'}
                    })
                    .then(resp=>{
                        if(resp.data.status == 'success'){
                            this.$message({
                                message: 'otp 已经发送到了您的手机上，请注意查收',
                                type: 'success'
                            });
                        }else{
                            this.$message.error('otp 发送失败，原因为：' + resp.data.data.errMsg);
                        }
                    })
                    .catch(err =>{
                        console.log();
                        this.$message.error('otp 发送失败，原因为：' + err.status + ', ' + err.statusText);
                    });
                },
            },
            
        });
    </script>

</html>
```


源码：[spring-boot-seckill](https://github.com/VictorBu/spring-boot-seckill)

