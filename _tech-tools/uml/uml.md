# UML

## Class Diagram
In UML class diagram, there are 6 relationships: 
- 泛化(Generalization)
- 实现(Realization)
- 关联(Association)
- 聚合(Aggregation)
- 组合(Composition)
- 依赖(Dependency)

### 泛化(Generalization)

【泛化关系】：是一种继承关系，表示一般与特殊的关系，它指定了子类如何特化父类的所有特征和行为。例如：老虎是动物的一种，即有老虎的特性也有动物的共性。

【箭头指向】：带三角箭头的实线，箭头指向父类

![generalization](generalization.png)

### 实现(Realization)

【实现关系】：是一种类与接口的关系，表示类是接口所有特征和行为的实现.

【箭头指向】：带三角箭头的虚线，箭头指向接口

![realization](realization.png)

### 关联(Association)

【关联关系】：是一种拥有的关系，它使一个类知道另一个类的属性和方法；如：老师与学生，丈夫与妻子关联可以是双向的，也可以是单向的。双向的关联可以有两个箭头或者没有箭头，单向的关联有一个箭头。

【代码体现】：成员变量

【箭头及指向】：带普通箭头的实心线，指向被拥有者

![association](association.png)

 上图中，老师与学生是双向关联，老师有多名学生，学生也可能有多名老师。但学生与某课程间的关系为单向关联，一名学生可能要上多门课程，课程是个抽象的东西他不拥有学生。 

 下图为自身关联： 

![association_1](association_1.png)

### 聚合(Aggregation)

【聚合关系】：是整体与部分的关系，且部分可以离开整体而单独存在。如车和轮胎是整体和部分的关系，轮胎离开车仍然可以存在。

聚合关系是关联关系的一种，是强的关联关系；关联和聚合在语法上无法区分，必须考察具体的逻辑关系。

【代码体现】：成员变量

【箭头及指向】：带空心菱形的实心线，菱形指向整体

![aggregation](aggregation.png)

### 组合(Composition)

【组合关系】：是整体与部分的关系，但部分不能离开整体而单独存在。如公司和部门是整体和部分的关系，没有公司就不存在部门。

组合关系是关联关系的一种，是比聚合关系还要强的关系，它要求普通的聚合关系中代表整体的对象负责代表部分的对象的生命周期。

【代码体现】：成员变量

【箭头及指向】：带实心菱形的实线，菱形指向整体

![composition](composition.png)

### 依赖(Dependency)

【依赖关系】：是一种使用的关系，即一个类的实现需要另一个类的协助，所以要尽量不使用双向的互相依赖.

【代码表现】：局部变量、方法的参数或者对静态方法的调用

【箭头及指向】：带箭头的虚线，指向被使用者

![dependency](dependency.png)

### 各种关系的强弱顺序

1. 泛化(Generalization) = 实现(Realization)
2. 组合(Composition)
3. 聚合(Aggregation)
4. 关联(Association)
5. 依赖(Dependency)

### Sample of Class Diagram

![class_diagram](class_diagram.png)

*其中，大雁和雁群的关系反了*

## References
- https://blog.csdn.net/zhangzeyuaaa/article/details/9207763