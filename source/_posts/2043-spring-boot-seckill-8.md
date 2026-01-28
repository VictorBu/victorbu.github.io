---
title: Spring Boot 构建电商基础秒杀项目 (八) 商品创建
date: 2019-03-21 17:00:00
updated: 2019-03-21 17:00:00
categories: [IT]
tags: [Java, Spring Boot]
---

> [SpringBoot构建电商基础秒杀项目](https://www.imooc.com/video/18390) 学习笔记


# 新建数据表

```
create table if not exists item (
    id int not null auto_increment,
    title varchar(64) not null default '',
    price double(10, 0) not null default 0,
    description varchar(500) null default '',
    sales int not null default 0,
    img_url varchar(200) null default '',
    primary key (id)
);

create table if not exists item_stock (
    id int not null auto_increment,
    stock int not null default 0,
    item_id int not null default 0,
    primary key (id)
);
```

# mybatis-generator.xml 添加

```
        <table tableName="item" domainObjectName="ItemDO"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false"></table>
        <table tableName="item_stock" domainObjectName="ItemStockDO"
               enableCountByExample="false" enableUpdateByExample="false"
               enableDeleteByExample="false" enableSelectByExample="false"
               selectByExampleQueryId="false"></table>
```

Run 'mybatis-generator'

# 新增 ItemModel

```
public class ItemModel {
    private Integer id;
    @NotBlank(message = "商品名称不能为空")
    private String title;
    @NotNull(message = "商品价格不能为空")
    @Min(value = 0, message = "商品价格必须大于0")
    private BigDecimal price;
    @NotNull(message = "库存不能为空")
    @Min(value = 0, message = "库存必须大于0")
    private Integer stock;
    @NotNull(message = "商品表述信息不能为空")
    private String description;
    private Integer sales;
    @NotNull(message = "商品图片不能为空")
    private String imgUrl;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        this.price = price;
    }

    public Integer getStock() {
        return stock;
    }

    public void setStock(Integer stock) {
        this.stock = stock;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public Integer getSales() {
        return sales;
    }

    public void setSales(Integer sales) {
        this.sales = sales;
    }

    public String getImgUrl() {
        return imgUrl;
    }

    public void setImgUrl(String imgUrl) {
        this.imgUrl = imgUrl;
    }
}
```

# 新增 ItemVO

```
public class ItemVO {
    private Integer id;
    private String title;
    private BigDecimal price;
    private Integer stock;
    private String description;
    private Integer sales;
    private String imgUrl;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        this.price = price;
    }

    public Integer getStock() {
        return stock;
    }

    public void setStock(Integer stock) {
        this.stock = stock;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public Integer getSales() {
        return sales;
    }

    public void setSales(Integer sales) {
        this.sales = sales;
    }

    public String getImgUrl() {
        return imgUrl;
    }

    public void setImgUrl(String imgUrl) {
        this.imgUrl = imgUrl;
    }
}

```

# 新增 ItemService

```
public interface ItemService {

    ItemModel createItem(ItemModel itemModel) throws BusinessException;

    List<ItemModel> listItem();

    ItemModel getItemById(Integer id);
}
```

# 新增 ItemServiceImpl

```
@Service
public class ItemServiceImpl implements ItemService {

    @Autowired
    private ValidatorImpl validator;

    @Autowired
    private ItemDOMapper itemDOMapper;

    @Autowired
    private ItemStockDOMapper itemStockDOMapper;

    @Override
    @Transactional
    public ItemModel createItem(ItemModel itemModel) throws BusinessException {

        ValidationResult result = validator.validate(itemModel);
        if(result.isHasErrors()){
            throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR, result.getErrMsg());
        }

        ItemDO itemDO = convertFromModel(itemModel);
        itemDOMapper.insertSelective(itemDO);

        itemModel.setId(itemDO.getId());

        ItemStockDO itemStockDO = convertItemStockFromModel(itemModel);
        itemStockDOMapper.insertSelective(itemStockDO);

        return getItemById(itemModel.getId());
    }

    @Override
    public List<ItemModel> listItem() {
        return null;
    }

    @Override
    public ItemModel getItemById(Integer id) {
        ItemDO itemDO = itemDOMapper.selectByPrimaryKey(id);
        if(itemDO == null){
            return null;
        }

        ItemStockDO itemStockDO = itemStockDOMapper.selectByItemId(itemDO.getId());

        ItemModel itemModel = convertFromDataObject(itemDO, itemStockDO);

        return itemModel;
    }

    private ItemDO convertFromModel(ItemModel itemModel){
        if(itemModel == null){
            return null;
        }

        ItemDO itemDO = new ItemDO();
        BeanUtils.copyProperties(itemModel, itemDO);

        itemDO.setPrice(itemModel.getPrice().doubleValue());

        return itemDO;
    }

    private ItemStockDO convertItemStockFromModel(ItemModel itemModel){
        if(itemModel == null){
            return null;
        }

        ItemStockDO itemStockDO = new ItemStockDO();
        itemStockDO.setItemId(itemModel.getId());
        itemStockDO.setStock(itemModel.getStock());

        return itemStockDO;
    }

    private ItemModel convertFromDataObject(ItemDO itemDO, ItemStockDO itemStockDO){
        if(itemDO == null){
            return null;
        }

        ItemModel itemModel = new ItemModel();
        BeanUtils.copyProperties(itemDO, itemModel);

        itemModel.setPrice(new BigDecimal(itemDO.getPrice()));

        itemModel.setStock(itemStockDO.getStock());

        return itemModel;
    }
}
```

# 新增 ItemController

```
@Controller("item")
@RequestMapping("/item")
@CrossOrigin(allowCredentials = "true", allowedHeaders = "*")
public class ItemController extends BaseController {

    @Autowired
    private ItemService itemService;

    @RequestMapping(value = "/create", method = {RequestMethod.POST}, consumes = {CONTENT_TYPE_FORMED})
    @ResponseBody
    public CommonReturnType createItem(@RequestParam(name="title") String title,
                                       @RequestParam(name="price") BigDecimal price,
                                       @RequestParam(name="description") String description,
                                       @RequestParam(name="stock") Integer stock,
                                       @RequestParam(name="imgUrl") String imgUrl) throws BusinessException {

        ItemModel itemModel = new ItemModel();
        itemModel.setTitle(title);
        itemModel.setPrice(price);
        itemModel.setDescription(description);
        itemModel.setStock(stock);
        itemModel.setImgUrl(imgUrl);

        itemModel = itemService.createItem(itemModel);

        ItemVO itemVO = convertFromModel(itemModel);

        return CommonReturnType.create(itemVO);
    }

    @RequestMapping(value = "/get", method = {RequestMethod.GET})
    @ResponseBody
    public CommonReturnType getItem(@RequestParam(name="id") Integer id){
        ItemModel itemModel = itemService.getItemById(id);

        ItemVO itemVO = convertFromModel(itemModel);

        return CommonReturnType.create(itemVO);
    }

    private ItemVO convertFromModel(ItemModel itemModel){
        if(itemModel == null){
            return null;
        }

        ItemVO itemVO = new ItemVO();
        BeanUtils.copyProperties(itemModel, itemVO);

        return itemVO;
    }
}
```

# 新增 createitem.html

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
                    <h3>创建商品</h3>
                    <el-form ref="form" :model="form" label-width="80px">
                        <el-form-item label="商品名">
                            <el-input v-model="form.title"></el-input>
                        </el-form-item>
                        <el-form-item label="商品描述">
                            <el-input v-model="form.description"></el-input>
                        </el-form-item>
                        <el-form-item label="价格">
                            <el-input v-model="form.price"></el-input>
                        </el-form-item>
                        <el-form-item label="图片">
                            <el-input v-model="form.imgUrl"></el-input>
                        </el-form-item>
                        <el-form-item label="库存">
                            <el-input v-model="form.stock"></el-input>
                        </el-form-item>
                        
                        <el-form-item>
                            <el-button type="primary" @click="onSubmit">提交</el-button>
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
                    title: '',
                    price: 0,
                    description: '',
                    stock: 1,
                    imgUrl: '',
                }
            },
            methods: {
                onSubmit(){
                
                    // https://www.cnblogs.com/yesyes/p/8432101.html
                    axios({
                        method: 'post',
                        url: 'http://localhost:8080/item/create',
                        data: this.form, 
                        params: this.form,
                        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                        withCredentials: true,
                    })
                    .then(resp=>{
                        if(resp.data.status == 'success'){
                            this.$message({
                                message: '创建成功',
                                type: 'success'
                            });
                        }else{
                            this.$message.error('创建失败，原因为：' + resp.data.data.errMsg);
                        }
                    })
                    .catch(err =>{
                        this.$message.error('创建失败，原因为：' + err.status + ', ' + err.statusText);
                    });
                },
            },
            
        });
    </script>

</html>
```


源码：[spring-boot-seckill](https://github.com/VictorBu/spring-boot-seckill)

