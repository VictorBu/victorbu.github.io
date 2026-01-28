---
title: Spring Boot 构建电商基础秒杀项目 (十一) 秒杀
date: 2019-03-24 07:00:00
updated: 2019-03-24 07:00:00
categories: [IT]
tags: [Java, Spring Boot]
---

> [SpringBoot构建电商基础秒杀项目](https://www.imooc.com/video/18390) 学习笔记

# 新建表

```
create table if not exists promo (
    id int not null auto_increment,
    promo_name varchar(64) not null default '',
    start_date datetime not null default '0000-00-00 00:00:00',
    end_date datetime not null default '0000-00-00 00:00:00',
    item_id int not null default 0,
    promo_item_price double(10,2) not null default 0,
    primary key (id)
);
```

# 新增 PromoModel

```
public class PromoModel {
    private Integer id;
    // 1: 还未开始 2: 进行中 3: 已结束
    private Integer status;
    private String promoName;
    private DateTime startDate;
    private DateTime endDate;
    private Integer itemId;
    private BigDecimal promoItemPrice;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Integer getStatus() {
        return status;
    }

    public void setStatus(Integer status) {
        this.status = status;
    }

    public String getPromoName() {
        return promoName;
    }

    public void setPromoName(String promoName) {
        this.promoName = promoName;
    }

    public DateTime getStartDate() {
        return startDate;
    }

    public void setStartDate(DateTime startDate) {
        this.startDate = startDate;
    }

    public DateTime getEndDate() {
        return endDate;
    }

    public void setEndDate(DateTime endDate) {
        this.endDate = endDate;
    }

    public Integer getItemId() {
        return itemId;
    }

    public void setItemId(Integer itemId) {
        this.itemId = itemId;
    }

    public BigDecimal getPromoItemPrice() {
        return promoItemPrice;
    }

    public void setPromoItemPrice(BigDecimal promoItemPrice) {
        this.promoItemPrice = promoItemPrice;
    }
}
```

# 新增 PromoService

```
public interface PromoService {
    PromoModel getPromoByItemId(Integer itemId);
}
```

# 新增 PromoServiceImpl

```
@Service
public class PromoServiceImpl implements PromoService {

    @Autowired
    private PromoDOMapper promoDOMapper;

    @Override
    public PromoModel getPromoByItemId(Integer itemId) {

        PromoDO promoDO = promoDOMapper.selectByItemId(itemId);

        PromoModel promoModel = convertFromDataObject(promoDO);

        if(promoDOMapper == null){
            return null;
        }

        if(promoModel.getStartDate().isAfterNow()){
            promoModel.setStatus(1);
        }else if(promoModel.getEndDate().isBeforeNow()){
            promoModel.setStatus(3);
        }else{
            promoModel.setStatus(2);
        }

        return promoModel;
    }

    private PromoModel convertFromDataObject(PromoDO promoDO){
        if(promoDO == null){
            return null;
        }

        PromoModel promoModel = new PromoModel();
        BeanUtils.copyProperties(promoDO, promoModel);

        promoModel.setPromoItemPrice(new BigDecimal(promoDO.getPromoItemPrice()));
        promoModel.setStartDate(new DateTime(promoDO.getStartDate()));
        promoModel.setEndDate(new DateTime(promoDO.getEndDate()));

        return promoModel;
    }
}
```

# ItemModel 添加

```
private PromoModel promoModel;
```

# 修改 ItemController, OrderController 对应的 service 及详情页

```
    private ItemVO convertFromModel(ItemModel itemModel){
        if(itemModel == null){
            return null;
        }

        ItemVO itemVO = new ItemVO();
        BeanUtils.copyProperties(itemModel, itemVO);

        if(itemModel.getPromoModel() != null){
            itemVO.setPromoStatus(itemModel.getPromoModel().getStatus());
            itemVO.setPromoId(itemModel.getPromoModel().getId());
            itemVO.setStartDate(itemModel.getPromoModel().getStartDate()
                    .toString(DateTimeFormat.forPattern("yyy-MM-dd HH:mm:ss")));
            itemVO.setPromoPrice(itemModel.getPromoModel().getPromoItemPrice());
        }else{
            itemVO.setPromoStatus(0);
        }

        return itemVO;
    }
```

```
@RequestMapping(value = "/createorder", method = {RequestMethod.POST}, consumes = {CONTENT_TYPE_FORMED})
    @ResponseBody
    public CommonReturnType createOrder(@RequestParam(name="itemId") Integer itemId,
                                        @RequestParam(name="amount") Integer amount,
                                        @RequestParam(name="promoId", required = false) Integer promoId)
            throws BusinessException {

        Boolean isLogin = (Boolean)httpServletRequest.getSession().getAttribute("LOGIN");
        if(isLogin == null || !isLogin.booleanValue()){
            throw new BusinessException(EmBusinessError.USER_NOT_LOGIN);
        }

        UserModel userModel = (UserModel)httpServletRequest.getSession().getAttribute("LOGIN_USER");

        OrderModel orderModel = orderService.createOrder(userModel.getId(), itemId, promoId, amount);

        return CommonReturnType.create(null);
    }
```


源码：[spring-boot-seckill](https://github.com/VictorBu/spring-boot-seckill)

