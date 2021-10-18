# （FSE/ESEC'21）Detecting Node.js Prototype Pollution Vulnerabilities via Object Lookup Analysis

原型链污染检测的论文，使用污点分析的方法。ObjLupAnsys,——一个流、上下文和分支敏感的静态污染分析工具

![image-20210915000403217](C:\Users\wwy\AppData\Roaming\Typora\typora-user-images\image-20210915000403217.png)

框架：

1. OBJLUPANSYS 打印出目标的AST，并构建一个特殊的图结构，名为Object Property Graph
2. 当所有的限制可以满足存在一个确定的污染路径，OBJLUPANSYS开始污点分析
3. ObjLupAnsys通过查询OPG来分析vulnerable对象