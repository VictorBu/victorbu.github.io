---
title: Spring Boot 构建电商基础秒杀项目 (五) 用户注册
date: 2019-03-19 07:00:00
updated: 2019-03-19 07:00:00
categories: [IT]
tags: [Java, Spring Boot]
---

> [SpringBoot构建电商基础秒杀项目](https://www.imooc.com/video/18390) 学习笔记

# UserService 添加

```
void register(UserModel userModel) throws BusinessException;
```

# UserServiceImpl 添加

```
    @Override
    @Transactional // 事务操作
    public void register(UserModel userModel) throws BusinessException {
        if(userModel == null){
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
        }

        if(StringUtils.isEmpty(userModel.getTelphone())
                || StringUtils.isEmpty(userModel.getName())
                || userModel.getGender() == null
                || userModel.getAge() == null
                || StringUtils.isEmpty(userModel.getEncrptPassword())){
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR);
        }

        UserDO userDO = convertFromModel(userModel);
        try{
            userDOMapper.insertSelective(userDO);
        }catch (DuplicateKeyException ex){
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR, "手机号码已注册");
        }

        userModel.setId(userDO.getId()); // 获取自增 id

        UserPasswordDO userPasswordDO = convertPasswordFromModel(userModel);
        userPasswordDOMapper.insertSelective(userPasswordDO);

        return;
    }

    private UserDO convertFromModel(UserModel userModel){
        if(userModel == null){
            return null;
        }

        UserDO userDO = new UserDO();
        BeanUtils.copyProperties(userModel, userDO);

        return userDO;
    }

    private UserPasswordDO convertPasswordFromModel(UserModel userModel){
        if(userModel == null){
            return null;
        }

        UserPasswordDO userPasswordDO = new UserPasswordDO();
        userPasswordDO.setEncrptPassword(userModel.getEncrptPassword());
        userPasswordDO.setUserId(userModel.getId());

        return userPasswordDO;
    }
```

# userDOMapper.xml 修改

返回自增 id

```
<insert id="insertSelective" parameterType="com.karonda.dataobject.UserDO" keyProperty="id" useGeneratedKeys="true">
```

# UserController 修改

```
@CrossOrigin(allowCredentials = "true", allowedHeaders = "*") // 解决跨域问题


    @RequestMapping(value = "/register", method = {RequestMethod.POST}, consumes = {CONTENT_TYPE_FORMED})
    @ResponseBody
    public CommonReturnType register(@RequestParam(name="telphone") String telphone,
                                     @RequestParam(name="otpCode") String otpCode,
                                     @RequestParam(name="name") String name,
                                     @RequestParam(name="gender") Integer gender,
                                     @RequestParam(name="age") Integer age,
                                     @RequestParam(name="password") String password)
            throws BusinessException, UnsupportedEncodingException, NoSuchAlgorithmException {

        String inSessionOtpCode = (String)this.httpServletRequest.getSession().getAttribute(telphone);
        if(!StringUtils.equals(otpCode, inSessionOtpCode)){
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR, "短信验证码不正确");
        }

        UserModel userModel = new UserModel();
        userModel.setTelphone(telphone);
        userModel.setName(name);
        userModel.setGender(new Byte(String.valueOf(gender)));
        userModel.setAge(age);
        userModel.setEncrptPassword(EncodeByMd5(password));
        userModel.setRegisterMode("byphone");

        userService.register(userModel);

        return CommonReturnType.create(null);
    }

    private String EncodeByMd5(String str) throws UnsupportedEncodingException, NoSuchAlgorithmException {
        MessageDigest md5 = MessageDigest.getInstance("MD5");
        BASE64Encoder base64Encoder = new BASE64Encoder();

        String newstr = base64Encoder.encode(md5.digest(str.getBytes("utf-8")));

        return newstr;
    }
```

# 新建 register.html

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
                    <h3>用户注册</h3>
                    <el-form ref="form" :model="form" label-width="80px">
                        <el-form-item label="手机号">
                            <el-input v-model="form.telphone"></el-input>
                        </el-form-item>
                        <el-form-item label="验证码">
                            <el-input v-model="form.otpCode"></el-input>
                        </el-form-item>
                        <el-form-item label="昵称">
                            <el-input v-model="form.name"></el-input>
                        </el-form-item>
                        <el-form-item label="性别">
                            <el-switch
                                v-model="form.gender"
                                active-color="#ff4949"
                                inactive-color="#13ce66"
                                active-value="2"
                                inactive-value="1"
                                active-text="女"
                                inactive-text="男"></el-switch>
                        </el-form-item>
                        <el-form-item label="年龄">
                            <el-input v-model="form.age"></el-input>
                        </el-form-item>
                        <el-form-item label="密码">
                            <el-input v-model="form.password" show-password></el-input>
                        </el-form-item>
                        
                        <el-form-item>
                            <el-button type="primary" @click="onSubmit">注册</el-button>
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
                    otpCode: '',
                    name: '',
                    gender: 1,
                    age: 0,
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
                        url: 'http://localhost:8080/user/register',
                        data: this.form, 
                        params: this.form,
                        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                        withCredentials: true,
                    })
                    .then(resp=>{
                        if(resp.data.status == 'success'){
                            this.$message({
                                message: '注册成功',
                                type: 'success'
                            });
                        }else{
                            this.$message.error('注册失败，原因为：' + resp.data.data.errMsg);
                        }
                    })
                    .catch(err =>{
                        this.$message.error('注册失败，原因为：' + err.status + ', ' + err.statusText);
                    });
                },
            },
            
        });
    </script>

</html>
```

**withCredentials 要设置为 true (getotp.html 同样需要设置)**

源码：[spring-boot-seckill](https://github.com/VictorBu/spring-boot-seckill)

