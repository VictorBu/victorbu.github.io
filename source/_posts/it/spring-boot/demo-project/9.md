---
title: Spring Boot 构建电商基础秒杀项目 (九) 商品列表 & 详情
date: 2019-03-22 07:00:00
updated: 2019-03-22 07:00:00
categories: [IT]
tags: [Java, Spring Boot]
---

> [SpringBoot构建电商基础秒杀项目](https://www.imooc.com/video/18390) 学习笔记

# ItemDOMapper.xml 添加

```
  <select id="listItem" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List" />
    from item
    order by sales desc
  </select>
```

# ItemDOMapper 添加

```
List<ItemDO> listItem();
```

# ItemServiceImpl 添加

```
    @Override
    public List<ItemModel> listItem() {
        List<ItemDO> itemDOList = itemDOMapper.listItem();

        List<ItemModel> itemModelList = itemDOList.stream().map(itemDO -> {
            ItemStockDO itemStockDO = itemStockDOMapper.selectByItemId(itemDO.getId());
            ItemModel itemModel = convertFromDataObject(itemDO, itemStockDO);
            return itemModel;
        }).collect(Collectors.toList());

        return itemModelList;
    }
```

# ItemController 添加

```
    @RequestMapping(value = "/list", method = {RequestMethod.GET})
    @ResponseBody
    public CommonReturnType listItem(){
        List<ItemModel> itemModelList = itemService.listItem();

        List<ItemVO> itemVOList = itemModelList.stream().map(itemModel -> {
            ItemVO itemVO = convertFromModel(itemModel);
            return itemVO;
        }).collect(Collectors.toList());

        return CommonReturnType.create(itemVOList);
    }
```

# 新建列表 & 详情页面

```
<html>
    <head>
        <meta charset="UTF-8">
        <link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
    </head>
    
    <body>
        <div id="app">
            <!--<item-list-->
                <!--v-for="item in items"-->
                <!--v-bind:item="item"-->
                <!--v-bind:key="item.id"></item-list>-->
            <template>
                <el-table
                        :data="items"
                        @row-click="handleClick"
                        stripe
                        style="width: 100%">
                    <el-table-column
                            prop="title"
                            label="商品名"
                            width="180">
                    </el-table-column>
                    <el-table-column
                            label="商品图片"
                            width="180">
                        <template slot-scope="scope">
                            <img :src="scope.row.imgUrl"  min-width="70" height="70" />
                        </template>
                    </el-table-column>
                    <el-table-column
                            prop="description"
                            label="商品描述">
                    </el-table-column>
                    <el-table-column
                            prop="price"
                            label="商品价格">
                    </el-table-column>
                    <el-table-column
                            prop="stock"
                            label="商品库存">
                    </el-table-column>
                    <el-table-column
                            prop="sales"
                            label="商品销量">
                    </el-table-column>
                </el-table>
            </template>
        </div>
    </body>
    
    <script src="https://unpkg.com/vue/dist/vue.js"></script>
    <script src="https://cdn.bootcss.com/axios/0.18.0/axios.min.js"></script>
    <script src="https://unpkg.com/element-ui/lib/index.js"></script>
    <script>
        // Vue.component('item-list',{
        //     props: ['item'],
        //     template:'<li>{{item.title}}</li>'
        // });
        var app = new Vue({
            el: '#app',
            data: {
                items: [],
            },
            methods: {
                getItems(){
                
                    // https://www.cnblogs.com/yesyes/p/8432101.html
                    axios({
                        method: 'get',
                        url: 'http://localhost:8080/item/list',
                        withCredentials: true,
                    })
                    .then(resp=>{
                        if(resp.data.status == 'success'){
                            this.items = resp.data.data;
                        }else{
                            this.$message.error('获取商品列表失败，原因为：' + resp.data.data.errMsg);
                        }
                    })
                    .catch(err =>{
                        this.$message.error('获取商品列表失败，原因为：' + err.status + ', ' + err.statusText);
                    });
                },
                handleClick(row){
                    window.location.href='getitem.html?id=' + row.id;
                },
            },
            mounted() {
                this.getItems()
            },
        });
    </script>

</html>
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
                    <h3>{{item.title}}</h3>
                    <el-form ref="form" :model="form" label-width="80px">
                        <el-form-item label="商品描述">
                            <label>{{item.description}}</label>
                        </el-form-item>
                        <el-form-item label="价格">
                            <label>{{item.price}}</label>
                        </el-form-item>
                        <el-form-item label="图片">
                            <img :src="item.imgUrl"  min-width="70" height="70" />
                        </el-form-item>
                        <el-form-item label="库存">
                            <label>{{item.stock}}</label>
                        </el-form-item>
                        <el-form-item label="销量">
                            <label>{{item.sales}}</label>
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
                item: {},
                form: {
                    id: 0,
                },
            enable: true,
            },
            methods: {
                getItem(){
                    this.form.id = this.getUrlKey("id");
                
                    // https://www.cnblogs.com/yesyes/p/8432101.html
                    axios({
                        method: 'get',
                        url: 'http://localhost:8080/item/get',
                        params: this.form,
                        withCredentials: true,
                    })
                    .then(resp=>{
                        if(resp.data.status == 'success'){
                            this.item = resp.data.data;
                        }else{
                            this.$message.error('获取商品失败，原因为：' + resp.data.data.errMsg);
                        }
                    })
                    .catch(err =>{
                        this.$message.error('获取商品失败，原因为：' + err.status + ', ' + err.statusText);
                    });
                },

                // https://www.cnblogs.com/xyyt/p/6068981.html
                getUrlKey(name){
                    return decodeURIComponent(
                        (new RegExp('[?|&]'+name+'='+'([^&;]+?)(&|#|;|$)')
                            .exec(location.href)||[,""])[1].replace(/\+/g,'%20'))||null;
                },
            },
            mounted() {
                this.getItem()
            },
        });
    </script>

</html>
```

源码：[spring-boot-seckill](https://github.com/VictorBu/spring-boot-seckill)

