---
title: Spring Boot 2.x 编写 RESTful API (五) 单元测试
date: 2019-04-08 07:00:00
updated: 2019-04-08 07:00:00
categories: [IT]
tags: [Java, Spring Boot, RESTful API, Unit testing]
---

> [用Spring Boot编写RESTful API](https://study.163.com/course/courseMain.htm?courseId=1005213034) 学习笔记


# 概念

## 驱动模块

## 被测模块

## 桩模块

+ 替代尚未开发完毕的子模块
+ 替代对环境依赖较大的子模块 (例如数据访问层)


# 示例

## 测试 Service

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class TvSeriesServiceTest {

    @MockBean
    TvSeriesDao tvSeriesDao;
    @MockBean
    TvCharacterDao tvCharacterDao;

    @Autowired
    TvSeriesService tvSeriesService;
    
    @Test
    public void testGetAll() {

        List<TvSeries> list = new ArrayList<>();
        TvSeries ts = new TvSeries();
        ts.setName("POI");
        list.add(ts);
        
        // 告诉 mock 当执行 getAll 方法时，返回上面创建的 list
        Mockito.when(tvSeriesDao.getAll()).thenReturn(list);

        List<TvSeries> result = tvSeriesService.getAllTvSeries();

        Assert.assertTrue(result.size() == list.size());
        Assert.assertTrue("POI".equals(result.get(0).getName()));   
    }
    
    @Test
    public void testGetOne() {
        //根据不同的传入参数，被 mock 的 bean 返回不同的数据的例子
        String newName = "Person Of Interest";
        BitSet mockExecuted = new BitSet();
        Mockito.doAnswer(new Answer<Object>() {
            @Override
            public Object answer(InvocationOnMock invocation) throws Throwable {
                Object[] args = invocation.getArguments();
                TvSeries bean = (TvSeries) args[0];
                //传入的值，应该和下面调用 tvSeriesService.updateTvSeries() 方法时的参数中的值相同
                Assert.assertEquals(newName, bean.getName());
                mockExecuted.set(0);
                return 1;
            }
        }).when(tvSeriesDao).update(any(TvSeries.class));
        
        TvSeries ts = new TvSeries();
        ts.setName(newName);
        ts.setId(111);
        
        tvSeriesService.updateTvSeries(ts);
        Assert.assertTrue(mockExecuted.get(0));
    }
}
```

## 测试 Controller

```
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc // 初始化一个 mvc 环境用于测试
public class TvSeriesControllerTest {

    @MockBean
    TvSeriesDao tvSeriesDao;
    @MockBean
    TvCharacterDao tvCharacterDao;

    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private TvSeriesController tvSeriesController;

    @Test
    public void testGetAll() throws Exception {
        List<TvSeries> list = new ArrayList<>();
        TvSeries ts = new TvSeries();
        ts.setName("POI");
        list.add(ts);

        Mockito.when(tvSeriesDao.getAll()).thenReturn(list);

        // 相当于在启动项目后，执行 GET /tvseries，被测模块是 web 控制层，因为 web 控制层会调用业务逻辑层，所以业务逻辑层也会被测试
        // 业务逻辑层调用了被 mock 出来的数据访问层桩模块。
        //如果想仅仅测试 web 控制层，（例如业务逻辑层尚未编码完毕），可以 mock 一个业务逻辑层的桩模块
        mockMvc.perform(MockMvcRequestBuilders.get("/tvseries"))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().string(Matchers.containsString("POI")));
    }
    
    @Test
    public void testAddSeries() throws Exception{
        BitSet bitSet = new BitSet(1);
        bitSet.set(0, false);

        Mockito.doAnswer(new Answer<Object>() {
            @Override
            public Object answer(InvocationOnMock invocation) throws Throwable {
                Object[] args = invocation.getArguments();
                TvSeries ts = (TvSeries) args[0];
                Assert.assertEquals(ts.getName(), "疑犯追踪");
                ts.setId(5432);
                bitSet.set(0, true);
                return 1;
            }
        }).when(tvSeriesDao).insert(Mockito.any(TvSeries.class));

        Mockito.doAnswer(new Answer<Object>() {
            @Override
            public Object answer(InvocationOnMock invocation) throws Throwable {
                Object[] args = invocation.getArguments();
                TvCharacter tc = (TvCharacter) args[0];
                // json 中传递过来的剧中角色名字
                Assert.assertEquals(tc.getName(), "芬奇");
                Assert.assertTrue(tc.getTvSeriesId() == 5432);
                bitSet.set(0, true);
                return 1;
            }
        }).when(tvCharacterDao).insert(Mockito.any(TvCharacter.class));
        
        String jsonData = "{\"name\":\"疑犯追踪\",\"seasonCount\":5,\"originRelease\":\"2011-09-22\",\"tvCharacters\":[{\"name\":\"芬奇\"}]}";
        this.mockMvc.perform(MockMvcRequestBuilders.post("/tvseries")
                .contentType(MediaType.APPLICATION_JSON).content(jsonData))
                .andDo(MockMvcResultHandlers.print())
                .andExpect(MockMvcResultMatchers.status().isOk());

        Assert.assertTrue(bitSet.get(0));
    }
    
    @Test
    public void testFileUpload() throws Exception{
        String fileFolder = "target/files/";
        File folder = new File(fileFolder);
        if(!folder.exists()) {
            folder.mkdirs();
        }
        // 设置 bean 里面通过 @Value 获得的数据
        ReflectionTestUtils.setField(tvSeriesController, "uploadFolder", folder.getAbsolutePath());
        
        InputStream is = getClass().getResourceAsStream("/testfileupload.jpg");
        if(is == null) {
            throw new RuntimeException("需要先在src/test/resources目录下放置一张jpg文件，名为testfileupload.jpg然后运行测试");
        }
        
        //模拟一个文件上传的请求
        MockMultipartFile imgFile = new MockMultipartFile("photo", "testfileupload.jpg", "image/jpeg", IOUtils.toByteArray(is));
        
        ResultActions result = mockMvc.perform(MockMvcRequestBuilders.multipart("/tvseries/1/photos")
                        .file(imgFile))
                        .andExpect(MockMvcResultMatchers.status().isOk());
        
        //解析返回的 JSON
        ObjectMapper mapper = new ObjectMapper();
        Map<String, Object> map = mapper.readValue(result.andReturn().getResponse().getContentAsString(), new TypeReference<Map<String, Object>>(){});
       
        String fileName = (String) map.get("photo");
        File f2 = new File(folder, fileName);
        //返回的文件名，应该已经保存在 filFolder 文件夹下
        Assert.assertTrue(f2.exists());
    }
}
```


源码：[spring-boot-2-restful](https://github.com/VictorBu/spring-boot-2-restful)