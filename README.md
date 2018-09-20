# jQueryWithReact

## 项目背景

项目背景：原项目是典型的MVC结构，前端页面的开发采用的是jQuery + ejs，通过NodeJS进行了路由跳转以及服务端渲染。

个人对jQuery用起来不是特别熟练，维护起来分外痛苦。其次，产品经理设计产品原型采用的是Ant Design，使用jQuery很难做到与产品设计的统一。

由于原项目已经很庞大，对项目代码进行重构不太现实，故只能针对后续需求的迭代开发采用React。

[三、webpack配置](https://github.com/Bian2017/jQueryWithReact/blob/master/doc/docs/%E4%B8%89%E3%80%81webpack%E9%85%8D%E7%BD%AE.md)

[四、性能优化](https://github.com/Bian2017/jQueryWithReact/blob/master/doc/docs/%E5%9B%9B%E3%80%81%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96.md)

[五、踩坑指南](https://github.com/Bian2017/jQueryWithReact/blob/master/doc/docs/%E4%BA%94%E3%80%81%E8%B8%A9%E5%9D%91%E6%8C%87%E5%8D%97.md)



六、待解决

在package.json中运行的编译脚本和在shell中输入相同的编译脚本，webpack生成的最终文件竟然不同。