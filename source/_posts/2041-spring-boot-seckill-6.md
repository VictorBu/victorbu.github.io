---
title: Spring Boot 构建电商基础秒杀项目 (六) 用户登陆
date: 2019-03-19 17:00:00
updated: 2019-03-19 17:00:00
categories: [IT]
tags: [Java, Spring Boot]
---

> [SpringBoot构建电商基础秒杀项目](https://www.imooc.com/video/18390) 学习笔记

# userDOMapper.xml 添加

```
  <select id="selectByTelphone" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List" />
    from user_info
    where telphone = #{telphone,jdbcType=VARCHAR}
  </select>
```

# userDOMapper 添加

```
    UserDO selectByTelphone(String telphone);
```

# UserService 添加

```
    UserModel validateLogin(String telphone, String encrptPassword) throws BusinessException;
```

# UserServiceImpl 添加

```
    @Override
    public UserModel validateLogin(String telphone, String encrptPassword) throws BusinessException {

        UserDO userDO = userDOMapper.selectByTelphone(telphone);

        if(userDO == null){
            throw new BusinessException(EmBusinessError.USER_LOGIN_FAIL);
        }

        UserPasswordDO userPasswordDO = userPasswordDOMapper.selectByUserId(userDO.getId());

        UserModel userModel = convertFromDataObject(userDO, userPasswordDO);

        if(!StringUtils.equals(encrptPassword, userModel.getEncrptPassword())){
            throw new BusinessException(EmBusinessError.USER_LOGIN_FAIL);
        }

        return userModel;
    }
```


# UserController 添加

```
    @RequestMapping(value = "/login", method = {RequestMethod.POST}, consumes = {CONTENT_TYPE_FORMED})
    @ResponseBody
    public CommonReturnType login(@RequestParam(name="telphone") String telphone,
                                     @RequestParam(name="password") String password)
            throws BusinessException, UnsupportedEncodingException, NoSuchAlgorithmException {

        if(StringUtils.isEmpty(telphone) || StringUtils.isEmpty(password)){
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
        }

        UserModel userModel = userService.validateLogin(telphone, EncodeByMd5(password));

        httpServletRequest.getSession().setAttribute("LOGIN", true);
        httpServletRequest.getSession().setAttribute("LOGIN_USER", userModel);

        return CommonReturnType.create(null);
    }
```

# 新建 login.html

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
                    <h3>用户登陆</h3>
                    <el-form ref="form" :model="form" label-width="80px">
                        <el-form-item label="手机号">
                            <el-input v-model="form.telphone"></el-input>
                        </el-form-item>
                        <el-form-item label="密码">
                            <el-input v-model="form.password" show-password></el-input>
                        </el-form-item>
                        
                        <el-form-item>
                            <el-button type="primary" @click="onSubmit">登陆</el-button>
                            <el-button @click="register">注册</el-button>
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
                    password: '',
                }
            },
            methods: {
                onSubmit(){
                
                    if(this.form.telphone == null || this.form.telphone == ''){
                        this.$message({
                            message: '手机号不能为空',
                            type: 'warning'
                        });
                        return;
                    }
                
                    // https://www.cnblogs.com/yesyes/p/8432101.html
                    axios({
                        method: 'post',
                        url: 'http://localhost:8080/user/login',
                        data: this.form, 
                        params: this.form,
                        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                        withCredentials: true,
                    })
                    .then(resp=>{
                        if(resp.data.status == 'success'){
                            this.$message({
                                message: '登陆成功',
                                type: 'success'
                            });
                        }else{
                            this.$message.error('登陆失败，原因为：' + resp.data.data.errMsg);
                        }
                    })
                    .catch(err =>{
                        this.$message.error('登陆失败，原因为：' + err.status + ', ' + err.statusText);
                    });
                },

                register(){
                    window.location.href='getotp.html';
                },

            },
            
        });
    </script>

</html>
```


源码：[spring-boot-seckill](https://github.com/VictorBu/spring-boot-seckill)

