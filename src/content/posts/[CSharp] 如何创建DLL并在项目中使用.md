---
title: '[C#] 如何创建DLL并在项目中使用'
slug: '20200705010533'
published: 2020-07-05T01:05:33
tags:
  - csharp
  - dotnet Framework
  - 类库 
  - 手册
category: '.NET'
description: '注意:文章适合初学者, 讲的较为详细, 大佬可以绕道作者也是自学C#的, 所以有些东西可能讲的也有些不好, 请见谅关于DLL:在C#中, DLL可以说是类库, 创建一个类库类型的项目后, 生成时生成的文件时一个DLL文件一个类库中, 包含一个或多个类, 这些类处于某个命名空间下, 当引用这个DLL文件后, using 相应的命名空间后即可直接使用类库中所包含的类一般的, 创建DLL文件为的是将自己定义的一个类, 直接制作成DLL文件以方便别人使用, 这样不需要复制代码也可以使用你定义的类.'
---

## 注意:

0. 文章适合初学者, 讲的较为详细, 大佬可以绕道
1. 作者也是自学C#的, 所以有些东西可能讲的也有些不好, 请见谅

## 关于DLL:

0. 在C#中, DLL可以说是类库, 创建一个类库类型的项目后, 生成时生成的文件时一个DLL文件
1. 一个类库中, 包含一个或多个类, 这些类处于某个命名空间下, 当引用这个DLL文件后, using 相应的命名空间后即可直接使用类库中所包含的类
2. 一般的, 创建DLL文件为的是将自己定义的一个类, 直接制作成DLL文件以方便别人使用, 这样不需要复制代码也可以使用你定义的类.
3. (对于不了解引用是什么的初学者, 你需要知道, 一个C#程序, 会使用不少的类库, 比如System.dll, System.Windows.Forms.dll, 但是这些.NET框架含有的类库, C#程序可以直接使用, 而不需要将DLL文件置于程序所在目录下, 项目的'引用'说明了这个程序使用了什么类库, 而且在你编程的时候, VS也会根据你引用的类库来给你代码提示)

## 创建DLL文件:

0. 创建一个 类库(.NET Framework) 项目, 如下图, 不过, 因为我已经有创建过了, 所以我就直接使用之前创建的项目了.

![创建一个 类库(.NET Framework) 项目](/images/2020070500212889.png)
1. 创建完成后, 你就可以写代码了, 要清楚, C#的类库包含的其实就是一个个的类, 或者其他成员(不能是字段).
2. 下图, 就是我之前的项目, 注意, 这些供使用者访问的成员别忘了使用public修饰. 
3. 我的这个项目, 是一个用来操作Json的类库, 之后将使用它来尝试操作DLL
![已经写好代码的类库项目](/images/20200705002655466.png)
4. 生成项目, 并找到生成的DLL文件.

![生成成功](/images/20200705003106729.png)
![这就是生成好的DLL文件](/images/20200705003205677.png)

## 引用生成的DLL:

0. 首先, 转到另外一个项目, 我们将在这个项目中使用之前生成的DLL.
![一个新的控制台项目](/images/20200705003504505.png)
1. 添加引用, 在解决方案管理器中右击项目, 添加, 引用,  
![添加引用](/images/20200705003754911.png)
2. 添加引用, 在弹出的窗口中点击浏览, 然后在选择文件窗口中选择之前生成的DLL文件, 点击添加, 最后在引用管理器中点击确定.

![添加引用](/images/20200705004309633.png)
3. 于是, 我们就成功的引用了之前生成的DLL文件.
![添加引用后](/images/20200705004359951.png)

## 使用DLL中包含的成员:

0. 在创建DLL时, 可以看到, 成员均处于 CHO.Json 命名空间下, 所以我们using它, 如下图, 并没有出现错误.
![添加引用](/images/20200705004541827.png)
1. 准备一个Json文本来使程序读取.


> {
    "姓名": "小明",
    "性别": "男",
    "年龄": 16,
    "自述": null,
    "是否患新冠肺炎": false,
    "学习的编程语言": ["C#", "Python", "C"]
}


2. 将其转换为字符串表达式.  (这里使用了我自己的小程序, 文章末尾会附上下载链接)
![转换为字符串表达式](/images/20200705005032983.png)
3. 将代码写好.

![使用类库中的成员](/images/20200705005246473.png)
4. 运行程序.
![程序运行结果](/images/20200705005450205.png)
5. 可以看到, 成功使用了类库中的JsonData类,和JsonDataType枚举类型.

## 文章到此结束.

0. 将文本转换为字符串表达式的小工具 ([蓝奏云下载](https://chonet.lanzous.com/i5Uveebcgle))
1. 关于我写的这个Json操作类库([文章](/posts/20201028235244/))

##### 关于作者:

0. 一个喜欢编程但不喜欢在校学习奇怪东西的奇怪人士.
1. 性格沙雕, 但写文章的时候就莫名其妙的认真起来
2. 喜欢玩MC, 还有类似的沙盒游戏, 例如废品工程师

##### 联系我:

0. 发送私信.
1. 电子邮件 : creepslime@foxmail.com
2. 私人QQ : 2056818509


欢迎与我交流吖   ε=ε=ε=(~ ￣▽￣)~
