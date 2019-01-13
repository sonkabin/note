# MyBatis plus

## 一些坑

- 有些版本不依赖mybatis-plus-generator，比如3.0.7.1。。。所有最好选择有依赖的，如3.0.4，否则自己添加依赖可能导致一个是3.x版本，一个是2.x版本

- 3.x版本中代码生成器配置自定义输出模板处，与2.x版本不同

  ```java
  3.x//若仍旧沿用2.x版本，不会在自己的resources下的templates查找，而是去它给的templates下查找
  TemplateConfig tc = new TemplateConfig();
  tc.setController("/mytemplates/controller.java");
  tc.setService("/mytemplates/service.java");
  tc.setServiceImpl("/mytemplates/serviceImpl.java");
  tc.setXml(null);//不让其同时在mapper包下生成xml文件
  mpg.setTemplate(tc);
  
  2.x//会在自己的resources下的templates查找
  TemplateConfig tc = new TemplateConfig();
  tc.setController("/templates/controller.java");
  tc.setService("/templates/service.java");
  tc.setServiceImpl("/templates/serviceImpl.java");
  tc.setXml(null);//不让其同时在mapper包下生成xml文件
  mpg.setTemplate(tc);
  ```

- 



