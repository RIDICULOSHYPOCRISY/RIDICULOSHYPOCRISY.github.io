---
title: c#_Delegate
date: 2021-11-08 15:48:24
tags:Csharp
---

# 委托（Delegate）

类似于c中的**指针**

方法的**参数是方法**

是存有对某个方法的引用的一种引用类型变量

引用可在运行时被改变

都派生自 **System.Delegate** 类

### 声明

###### 基本语法

```c#
delegate <return type> <delegate_name> <parameter list>
```

###### e.g.

```c#
public delegate int MyDelegate (string s);
```

上面的委托可被用于引用任何一个带有一个单一的 *string* 参数的方法，并返回一个 *int* 类型变量。

### 实例化

委托对象必须使用 **new** 关键字来创建

与一个特定方法有关

当创建委托时，传递到 **new** 语句的参数就像方法调用一样书写，但是不带有参数

###### e.g.

```c#
MyDelegate example1 = new MyDelegate(Method1);
MyDelegate example2 = new MyDelegate(Method2);

```

### 委托的多播

委托对象可使用 "+" 运算符进行合并，“-”移除

只有相同类型的委托可被合并

可以创建一个委托被调用时要调用的方法的调用列表

```c#
using System;

delegate int NumberChanger(int n);
namespace
{
   class
   {
      static int num = 10;
      public static int AddNum(int p)
      {
         ...
         return num;
      }

      public static int MultNum(int q)
      {
         ...
         return num;
      }
      public static int getNum()
      {
         return num;
      }

      static void Main(string[] args)
      {
         // 创建委托实例
         NumberChanger nc;
         NumberChanger nc1 = new NumberChanger(AddNum);
         NumberChanger nc2 = new NumberChanger(MultNum);
         nc = nc1;
         nc += nc2;
         // 调用多播
         nc(5);
         Console.WriteLine("Value of Num: {0}", getNum());
         Console.ReadKey();
      }
   }
}
```

