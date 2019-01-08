---
layout: mypost
title: 集成Activiti Modeler
categories: [Activiti]
---

#### **1.下载源文件**

[activiti-5.22.0官方Demo](https://pan.baidu.com/s/1Lv3zSBpPNW4d6M6DXPsoTw)

[activiti5.22.0源码](https://pan.baidu.com/s/1xPptjjNnbMf_2XSrHcKcPA)

#### **2.copy源文件**
 * （一）复制前端文件

> 解压activiti-5.22.0官方Demo

![](https://wx1.sinaimg.cn/mw690/becbe214gy1fq6ion7ue2j20eu048dfv.jpg)

> 解压activiti-explorer.war

![](https://wx4.sinaimg.cn/mw690/becbe214gy1fq6ip9yo0vj20d405ejrj.jpg)

> 复制editor-app,diagram-viewer文件夹，以及modele.html到本地项目

![](https://wx3.sinaimg.cn/mw690/becbe214gy1fq6imxshnpj20b70e9jrw.jpg)

*  （2）复制服务端文件

> 解压activiti5.22.0源码

![](https://wx4.sinaimg.cn/mw690/becbe214gy1fq6j9d4wzgj20vu0hfq49.jpg)

> 复制ModelEditorJsonRestResource.java,ModelSaveRestResource.java,StencilsetRestResource.java到控制层文件夹下

![](https://wx1.sinaimg.cn/mw690/becbe214gy1fq7brr7hyqj20d10gfq3k.jpg)

#### **3.添加maven引用**
```xml
<dependency>
	<groupId>org.activiti</groupId>
	<artifactId>activiti-modeler</artifactId>
	<version>5.22.0</version>
</dependency>
```
#### **4.启动项目,访问modeler页面**

[http://localhost:8080/modeler.html](http://localhost:8080/modeler.html)

![这里写图片描述](https://wx1.sinaimg.cn/mw690/becbe214gy1fq7c0o0nvcj209o06baa0.jpg)

> 修改启动类，屏蔽登录功能

```java
@SpringBootApplication
@EnableAutoConfiguration(exclude = {
		org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration.class,
		org.activiti.spring.boot.SecurityAutoConfiguration.class,
})
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```
>再次访问modeler页面

![这里写图片描述](https://wx4.sinaimg.cn/mw690/becbe214gy1fq7cah1v0qj21f80sa417.jpg)

>处理 `/activiti-explorer/service/model//json` 请求的报错

该请求的控制类为`ModelEditorJsonRestResource.java`

```
@RequestMapping(value="/model/{modelId}/json", method = RequestMethod.GET, produces = "application/json")
```
在`/public/editor-app/app-cfg.js`文件中修改请求的地址

```js
ACTIVITI.CONFIG = {
	//'contextRoot' : '/activiti-explorer/service',
	'contextRoot' : '',
};
```

> 再次访问modeler页面

报错`GET http://localhost:8080/model//json 404 ()`
这是因为我们没有已经建好的model模型，无法查看

#### **5.新建model**
> 测试类
```
@Controller
@RequestMapping("model")
public class ModelTest {

    @RequestMapping("create")
    public void createModel(HttpServletRequest request, HttpServletResponse response){
        try{
            String modelName = "modelName";
            String modelKey = "modelKey";
            String description = "description";

            ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

            RepositoryService repositoryService = processEngine.getRepositoryService();

            ObjectMapper objectMapper = new ObjectMapper();
            ObjectNode editorNode = objectMapper.createObjectNode();
            editorNode.put("id", "canvas");
            editorNode.put("resourceId", "canvas");
            ObjectNode stencilSetNode = objectMapper.createObjectNode();
            stencilSetNode.put("namespace", "http://b3mn.org/stencilset/bpmn2.0#");
            editorNode.put("stencilset", stencilSetNode);
            Model modelData = repositoryService.newModel();

            ObjectNode modelObjectNode = objectMapper.createObjectNode();
            modelObjectNode.put(ModelDataJsonConstants.MODEL_NAME, modelName);
            modelObjectNode.put(ModelDataJsonConstants.MODEL_REVISION, 1);
            modelObjectNode.put(ModelDataJsonConstants.MODEL_DESCRIPTION, description);
            modelData.setMetaInfo(modelObjectNode.toString());
            modelData.setName(modelName);
            modelData.setKey(modelKey);

            //保存模型
            repositoryService.saveModel(modelData);
            repositoryService.addModelEditorSource(modelData.getId(), editorNode.toString().getBytes("utf-8"));
            response.sendRedirect(request.getContextPath() + "/modeler.html?modelId=" + modelData.getId());
        }catch (Exception e){
        }
    }

}
```
> 访问`localhost:8080/model/create`

![这里写图片描述](https://wx2.sinaimg.cn/mw690/becbe214gy1fq7dx00vb8j21h70sg0w4.jpg)

>处理`http://localhost:8080/editor/stencilset?version=1523329753442`请求的报错

在StencilsetRestResource.java中，我们项目中少了stencilset.json
```java
@RestController
public class StencilsetRestResource {

  @RequestMapping(value="/editor/stencilset", method = RequestMethod.GET, produces = "application/json;charset=utf-8")
  public @ResponseBody String getStencilset() {
  //stencilset.json为Model中的工具栏的名称字符，这里在resources下面查找
    InputStream stencilsetStream = this.getClass().getClassLoader().getResourceAsStream("stencilset.json");
    try {
      return IOUtils.toString(stencilsetStream, "utf-8");
    } catch (Exception e) {
      throw new ActivitiException("Error while loading stencil set", e);
    }
  }
}
```
> 下载[stencilset.json](https://pan.baidu.com/s/1RV-71YHWnYxWINO5mdUZbA)，并放置在resources目录下
>  再次访问`localhost:8080/model/create`

#### **6.保存Model**
![这里写图片描述](https://wx3.sinaimg.cn/mw690/becbe214gy1fq7eb4mxzsj21h90qpjuy.jpg)

> 处理`http://localhost:8080/model/102503/save`的报错

修改`ModelSaveRestResource.java`为

```
@RestController
public class ModelSaveRestResource implements ModelDataJsonConstants {

  protected static final Logger LOGGER = LoggerFactory.getLogger(ModelSaveRestResource.class);

  @Autowired
  private RepositoryService repositoryService;

  @Autowired
  private ObjectMapper objectMapper;

  @RequestMapping(value="/model/{modelId}/save", method = RequestMethod.PUT)
  @ResponseStatus(value = HttpStatus.OK)
  public void saveModel(@PathVariable String modelId, String name, String description, String json_xml, String svg_xml) {
    try {

      Model model = repositoryService.getModel(modelId);

      ObjectNode modelJson = (ObjectNode) objectMapper.readTree(model.getMetaInfo());

      modelJson.put(MODEL_NAME, name);
      modelJson.put(MODEL_DESCRIPTION, description);
      model.setMetaInfo(modelJson.toString());
      model.setName(name);

      repositoryService.saveModel(model);

      repositoryService.addModelEditorSource(model.getId(), json_xml.getBytes("utf-8"));

      InputStream svgStream = new ByteArrayInputStream(svg_xml.getBytes("utf-8"));
      TranscoderInput input = new TranscoderInput(svgStream);

      PNGTranscoder transcoder = new PNGTranscoder();
      // Setup output
      ByteArrayOutputStream outStream = new ByteArrayOutputStream();
      TranscoderOutput output = new TranscoderOutput(outStream);

      // Do the transformation
      transcoder.transcode(input, output);
      final byte[] result = outStream.toByteArray();
      repositoryService.addModelEditorSourceExtra(model.getId(), result);
      outStream.close();

    } catch (Exception e) {
      LOGGER.error("Error saving model", e);
      throw new ActivitiException("Error saving model", e);
    }
  }
}
```
> 再次保存，保存成功。Activiti Modeler集成成功

* [源码下载](https://pan.baidu.com/s/1B2QUsMIaOiUegEHaKvzOzw)
