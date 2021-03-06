Integrating React into Existing jQuery Web Applications
---

### 项目背景

原项目是典型的MVC结构，前端页面开发采用的是jQuery + EJS，通过NodeJS进行了路由跳转以及中间层处理。

个人对jQuery用起来不是特别熟练，维护起来比较痛苦。其次，产品经理设计产品原型采用的是蚂蚁金服的Ant Design，使用jQuery + Bootstrap很难做到与产品原型的统一。

由于我个人对React比较熟练，就想在原项目的基础上引入React框架。而原项目比较庞大，短期内对项目代码进行重构不太现实，故只能针对产品后续需求开发采用React。在使用React开发后续需求的过程中，遇到了不少坑，针对这些坑的处理，整理了以下文档，如下所示：

[一、atool-build使用小结以及弃坑缘由](https://github.com/Bian2017/jQueryWithReact/blob/master/doc/docs/%E4%B8%80%E3%80%81%08atool-build%E4%BD%BF%E7%94%A8%E5%B0%8F%E7%BB%93%E4%BB%A5%E5%8F%8A%E5%BC%83%E5%9D%91%E7%BC%98%E7%94%B1.md)

[二、webpack配置](https://github.com/Bian2017/IntegratingReactIntoJqueryWeb/blob/master/doc/docs/%E4%BA%8C%E3%80%81webpack%E9%85%8D%E7%BD%AE.md)

[三、性能优化](https://github.com/Bian2017/IntegratingReactIntoJqueryWeb/blob/master/doc/docs/%E4%B8%89%E3%80%81%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.md)

[四、React使用小结](https://github.com/Bian2017/IntegratingReactIntoJqueryWeb/blob/master/doc/docs/%E5%9B%9B%E3%80%81React%E4%BD%BF%E7%94%A8%08%E5%B0%8F%E7%BB%93.md)

[五、Ant Design使用小结](https://github.com/Bian2017/IntegratingReactIntoJqueryWeb/blob/master/doc/docs/%E4%BA%94%E3%80%81Ant%20Desgin%E4%BD%BF%E7%94%A8%E5%B0%8F%E7%BB%93.md)

[六、jQuery使用小结](https://github.com/Bian2017/IntegratingReactIntoJqueryWeb/blob/master/doc/docs/%E5%85%AD%E3%80%81jQuery%E4%BD%BF%E7%94%A8%E5%B0%8F%E7%BB%93.md)