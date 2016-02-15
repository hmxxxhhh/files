# 构建
## 目录
------
* [理想设计](#理想设计)
* [代码技巧](#代码技巧)
* [设计流程](#设计流程)


## 理想设计
------
* 最小复杂度：分解多个子系统降低问题的复杂度、且设计短小的子程序。
* 易于维护：写代码也要替你的读者想想，赋予可读性。读代码时间要比写代码的时间要多，且不同维护人来看，所以此见事情很重要。
* 松散耦合：低耦合，尽量与其他不相关，独立运作。相互依赖少。水管概念，连接越少装修就越容易装修。
* 高扇入：大量的类使用，如utility classes。把相关操作包到一个 utility class
* 低扇出：一个类里少量用其他的类。最好低于七个。不然过于复杂。（应用越多错误率越高）
* 可扩展：改动某个地方而不会影响。
* 可重用性：最好供给大家一起共用，小少代码重复。
* 可移植性
* 精简性：去除不必要的代码与注解。
* 层次性：中间层，负责新旧交替与封装概念。


## 代码技巧
------
### 设计阶段
* 多利用画图解释
* TFD(test-first Development)。
* 创建中央控制点，尽量在同一个地方找到某样事务，相对安全容易。
* 信息统一，100 -> 定义为 MAX_EMPLOYEES 。改一个地方全部都改。没有重复的代码。
* 达到信息隐藏，不必要的不显示于外部。分为 public、private。不要因为一个类需要用到就当成公开接口；若是private设定错为public，有可能会误导以为外面可以设置，增加维护成本。
* 类内的数据，不应该定义为全局。
* 避免循环依赖（retain cycle）
* 对象参数耦合：ObjectA -> ObjectB 若很多参数都需要设定，考虑 ObjectC 设计。传递 Object
* 高内聚：一个中心目逼上的紧密合作。（高内聚 50%没有错误，低内聚 18%没有错误，功能内聚力来说： sin()）
* method 有 switch case，可能需要用到继承来实作。看情况而定。

### 子程序
* 命名要有意义、不要数字part1 part2..NO
* 缩小代码规模 50~150行，便于查找问题、优化。(子程序长度与错误率成反比)
* 子程序不能修改请定义 const or final。
* 正确定义宏 macro #dfine Cub(a) ((a)*(a)*(a))
* 多定义枚举，比 switch(type) typea: 比 switch(type) 0: 更清楚。

![Alt text](https://raw.githubusercontent.com/MingYi-Chen/sharedPhotoes/master/enum.png)

* 参数：少于7个；传入与定义的参数就一定要用，避免阅读障碍
* 输入值，必须清楚知道其范围。
* 防御式子程序：比如除以零...等等。或有可能造成crash的可能。
* 错误处理方式：返回错误代码(错误代码以怎样显示给用户？)、或error写于日志传于Server(测试时候不应该出现的条件，断言直接crash)、严重的问题以关闭程序如交易。

### 变量
* 命名，有意义 9-16 字符

![Alt text](https://raw.githubusercontent.com/MingYi-Chen/sharedPhotoes/master/loop_value.png)

* 存活时间

![Alt text](https://raw.githubusercontent.com/MingYi-Chen/sharedPhotoes/master/value_scope0.png)

![Alt text](https://raw.githubusercontent.com/MingYi-Chen/sharedPhotoes/master/value_scope2.png) 变成
![Alt text](https://raw.githubusercontent.com/MingYi-Chen/sharedPhotoes/master/value_scope1.png)

如下

![Alt text](https://raw.githubusercontent.com/MingYi-Chen/sharedPhotoes/master/value_scope3.png)

* 作用域。越小越好。减少用全局数据。
* 不要有 magic number。129.1000...等等的 hard code。
* 一个变量单一用途，有时候会 temp 从上面用到下面。不好的作法
* 类第一个字符大写，变量都小写，如 Widget widget;
* 变量不要有数字，width1 width2

### if or (switch case)
* if( 'a' >= inputCharacter && 'z' <= inputCharacter) -> if(IsLetter(inputCharacter)) 更容易读。
* 正常情况放前面，if(status == sucessful)
* switch case 频率越高放于上面 或 按照字母

## 其他
* 之前看人开发，有把 doc 文件放于 eclipse 上面。编译上不编译。这样会达到不错的效益吗？
* 设计实践遇到问题：爱迪生 1000次失败浪费，已经发现1000种材料不能用。

## 设计流程
------
### 阶段性分解

![Alt text](https://raw.githubusercontent.com/MingYi-Chen/sharedPhotoes/master/level_design.png)

### 类的设计流程
![Alt text](https://raw.githubusercontent.com/MingYi-Chen/sharedPhotoes/master/design_class.png)

* 创建类
* 创建子程序
* 审查测试类

### 子程序的设计流程
![Alt text](https://raw.githubusercontent.com/MingYi-Chen/sharedPhotoes/master/design_routine.png)

* 设计子程序。定义子程序解决之问题。
* 决定如何测试子程序。
* 检查设计
* 编写子程序代码
* 审查测试子程序

### 编辑子程序
![Alt text](https://raw.githubusercontent.com/MingYi-Chen/sharedPhotoes/master/routine_process.png)

* 写子程序声明
* 写伪代码 -> 变成注释 -> 填充代码
* 检查代码
* 收尾 -> 完成。

update time：2016-01-10
