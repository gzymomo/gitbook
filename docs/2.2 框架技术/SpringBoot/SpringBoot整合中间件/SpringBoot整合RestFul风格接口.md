[TOC]

# 2、RestFul 风格接口和测试
## 2.1 Rest风格接口
```java
/**
 * Rest 风格接口测试
 */
@RestController // 等价 @Controller + @ResponseBody 返回Json格式数据
@RequestMapping("rest")
public class RestApiController {
    private static final Logger LOG = LoggerFactory.getLogger(RestApiController.class) ;
    /**
     * 保存
     */
    @RequestMapping(value = "/insert",method = RequestMethod.POST)
    public String insert (UserInfo userInfo){
        LOG.info("===>>"+userInfo);
        return "success" ;
    }
    /**
     * 查询
     */
    @RequestMapping(value = "/select/{id}",method = RequestMethod.GET)
    public String select (@PathVariable Integer id){
        LOG.info("===>>"+id);
        return "success" ;
    }
}
```
## 2.2 测试代码
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = MockServletContext.class)
@WebAppConfiguration
public class TestRestApi {

    private MockMvc mvc;
    @Before
    public void setUp() throws Exception {
        mvc = MockMvcBuilders.standaloneSetup(new RestApiController()).build();
    }

    /**
     * 测试保存接口
     */
    @Test
    public void testInsert () throws Exception {
        RequestBuilder request = null;
        request = post("/rest/insert/")
                .param("id", "1")
                .param("name", "测试大师")
                .param("age", "20");
        mvc.perform(request)
                .andExpect(content().string(equalTo("success")));
    }

    /**
     * 测试查询接口
     */
    @Test
    public void testSelect () throws Exception {
        RequestBuilder request = null;
        request = get("/rest/select/1");
        mvc.perform(request)
                .andExpect(content().string(equalTo("success")));
    }
}
```