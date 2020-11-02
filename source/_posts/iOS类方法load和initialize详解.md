
---
title: iOS类方法load和initialize详解
date: 
tags:
---

iOS开发中总能看到+load和+initialize的身影,网上对于这两个方法有很多解释,官方也有说明,但有些细节不够清楚,今天我们来详细扒一扒这两个方法.

## load

Apple文档是这样描述的

```
Invoked whenever a class or category is added to the Objective-C runtime; implement this method to perform class-specific behavior upon loading.

当类（Class）或者类别（Category）加入Runtime中时（就是被引用的时候）。
实现该方法，可以在加载时做一些类特有的操作。

Discussion

The load message is sent to classes and categories that are both dynamically loaded and statically linked, but only if the newly loaded class or category implements a method that can respond.

The order of initialization is as follows:

All initializers in any framework you link to.
调用所有的Framework中的初始化方法

All +load methods in your image.
调用所有的+load方法

All C++ static initializers and C/C++ attribute(constructor) functions in your image.
调用C++的静态初始化方及C/C++中的attribute(constructor)函数

All initializers in frameworks that link to you.
调用所有链接到目标文件的framework中的初始化方法

In addition:

A class’s +load method is called after all of its superclasses’ +load methods.
一个类的+load方法在其父类的+load方法后调用

A category +load method is called after the class’s own +load method.
一个Category的+load方法在被其扩展的类的自有+load方法后调用

In a custom implementation of load you can therefore safely message other unrelated classes from the same image, but any load methods implemented by those classes may not have run yet.
在+load方法中，可以安全地向同一二进制包中的其它无关的类发送消息，但接收消息的类中的+load方法可能尚未被调用。

```

### load函数调用特点如下:

当类被引用进项目的时候就会执行load函数(在main函数开始执行之前）,与这个类是否被用到无关,每个类的load函数只会自动调用一次.由于load函数是系统自动加载的，因此不需要调用父类的load函数，否则父类的load函数会多次执行。

1.当父类和子类都实现load函数时,父类的load方法执行顺序要优先于子类

2.当子类未实现load方法时,不会调用父类load方法

3.类中的load方法执行顺序要优先于类别(Category)

4.当有多个类别(Category)都实现了load方法,这几个load方法都会执行,但执行顺序不确定(其执行顺序与类别在Compile Sources中出现的顺序一致)

5.当然当有多个不同的类的时候,每个类load 执行顺序与其在Compile Sources出现的顺序一致

### 下面通过实例来一起验证下:

我们新建2个类:Person继承NSObject,Student和Teacher均继承Person

```

Person : NSObject
Student : Person
Teacher : Person

```
新建3个Person分类:

```
Person (Category)
Person (Category2)
Person (Category3)

```

在Person,Student,Person (Category),Person (Category2),Person (Category3)的.m文件中实现下面方法(Teacher除外)

```
//In Person.m
+(void)load
{
    NSLog(@"%s",__FUNCTION__);
}
//In Student.m(继承自Person)
+(void)load
{
    NSLog(@"%s",__FUNCTION__);
}

//In Teacher.m(继承Person)
不实现load方法

//In Person+Category.m
+(void)load
{
    NSLog(@"%s",__FUNCTION__);
}
//In Person+Category2.m
+(void)load
{
    NSLog(@"%s",__FUNCTION__);
}
//In Person+Category3.m
+(void)load
{
    NSLog(@"%s",__FUNCTION__);
}

```

运行结果:

```

2020-10-04 11:11:40.612 LoadAndInitializeExample[27745:856244] +[Person load]
2020-10-0411:11:40.615 LoadAndInitializeExample[27745:856244] +[Student load]
2020-10-04 11:11:40.616 LoadAndInitializeExample[27745:856244] +[Person(Category3) load]
2020-10-04 11:11:40.617 LoadAndInitializeExample[27745:856244] +[Person(Category) load]
2020-10-04 11:11:40.617 LoadAndInitializeExample[27745:856244] +[Person(Category2) load]

```

可以看到执行顺序依次为:

1.首先执行的是父类Person load方法,再执行的是子类Student load方法,说明父类的load方法执行顺序要优先于子类

2.子类Teacher中没有实现load方法,没有打印,说明子类没有实现load方法时并不会调用父类load方法

3.最后执行的是Person 3个Category的load方法,并且没有顺序,说明类别(Category)中的load方法要晚于类,多个类别(Category)都实现了load方法,这几个load
方法都会执行,但执行顺序不确定

4.同时我们也可以看到这几个Category load 执行顺序与其在Compile Sources中出现的顺序一致

5.当然多个不同的类 其load执行顺序,也与其在Compile Sources出现的顺序一致

## initialize

Apple文档是这样描述的

```
Initializes the class before it receives its first message.

在这个类接收第一条消息之前调用。

Discussion

The runtime sends initialize to each class in a program exactly one time just before the class, or any class that inherits from it, is sent its first message from within the program. (Thus the method may never be invoked if the class is not used.) The runtime sends the initialize message to classes in a thread-safe manner. Superclasses receive this message before their subclasses.

Runtime在一个程序中每一个类的一个程序中发送一个初始化一次，或是从它继承的任何类中，都是在程序中发送第一条消息。（因此，当该类不使用时，该方法可能永远不会被调用。）运行时发送一个线程安全的方式初始化消息。父类的调用一定在子类之前。


```

### initialize函数调用特点如下:

initialize在类或者其子类的第一个方法被调用前调用。即使类文件被引用进项目,但是没有使用,initialize不会被调用。由于是系统自动调用，也不需要再调用  [super initialize] ，否则父类的initialize会被多次执行。假如这个类放到代码中，而这段代码并没有被执行，这个函数是不会被执行的。

1.父类的initialize方法会比子类先执行

2.当子类未实现initialize方法时,会调用父类initialize方法,子类实现initialize方法时,会覆盖父类initialize方法.

3.当有多个Category都实现了initialize方法,会覆盖类中的方法,只执行一个(会执行Compile Sources 列表中最后一个Category 的initialize方法)

### 我们同样通过实例来一起验证下:

我们添加Person类,在.m中实现+ initialize方法

```
//In Person.m
+(void)initialize
{
     NSLog(@"%s",__FUNCTION__);
}


```
运行结果:

```
无打印...
```

说明:只是把类文件被引用进项目,没有使用的话,initialize不会被调用

我们在Teacher(继承Person).m中不实现initialize方法

```

//In Person.m
+(void)initialize
{
     NSLog(@"%s",__FUNCTION__);
}

//In Teacher.m(继承自Person)

Teacher.m中不实现initialize方法


```

我们调用下Teacher的new方法,运行

```

[Teacher new];


```

运行结果:

```
2020-10-04 14:06:45.051 LoadAndInitializeExample[29238:912579] +[Person initialize]
2020-10-04 14:06:45.051 LoadAndInitializeExample[29238:912579] +[Person initialize]

```

运行后发现父类的initialize方法竟然调用了两次：

可见当子类未实现initialize方法,会调用父类initialize方法.

但为什么会打印2次呢?

我的理解:

子类不实现initialize方法，会把继承父类的initialize方法并调用一遍。在此之前,父类初始化时,会先调用一遍自己initialize方法.所以出现2遍,所以为了防止父类initialize中代码多次执行,我们应该这样写:

```
//In Person.m
+(void)initialize
{
	if(self == [Person class])
	{
          NSLog(@"%s",__FUNCTION__);
	}
}


```
下面我们在 Teacher.m中实现initialize方法

```
//In Person.m
+(void)initialize
{
     NSLog(@"%s",__FUNCTION__);
}

//In Teacher.m(继承自Person)
+(void)initialize
{
     NSLog(@"%s",__FUNCTION__);
}

```

同样调用Teacher 的new方法,运行

```

[Teacher new];


```

运行结果:

```
2020-10-04 14:07:45.051 LoadAndInitializeExample[29238:912579] +[Person initialize]
2020-10-04 14:07:45.051 LoadAndInitializeExample[29238:912579] +[Teacher initialize]

```

可以看到 当子类实现initialize方法后,会覆盖父类initialize方法,这一点和继承思想一样

我们在Person.m和Person+Category.m中实现initialize方法

```
//In Person.m
+(void)initialize
{
     NSLog(@"%s",__FUNCTION__);
}

//In Person+Category.m
+(void)initialize
{
     NSLog(@"%s",__FUNCTION__);
}


```

调用Person 的 new方法,运行

```
[Person new];

```

运行结果:

```
2020-10-05 09:46:51.054 LoadAndInitializeExample[38773:1226306] +[Person(Category) initialize]

```

运行后可以看到Person的initialize方法并没有被执行,已经被Person+Category中的initialize取代了
当有多个Category时会怎样了,我们在Person,Person+Category,Person+Category2,Person+Category3 的.m中都实现initialize方法

```
//In Person.m
+(void)initialize
{
     NSLog(@"%s",__FUNCTION__);
}

//In Person+Category.m
+(void)initialize
{
     NSLog(@"%s",__FUNCTION__);
}
//In Person+Category2.m
+(void)initialize
{
     NSLog(@"%s",__FUNCTION__);
}
//In Person+Category3.m
+(void)initialize
{
     NSLog(@"%s",__FUNCTION__);
}

```

可以看到,当存在多个Category时,也只执行一个,具体执行哪一个Category中的initialize方法,测试后便可发现,会执行Compile Sources 列表中最后一个Category 的initialize方法

## 什么情况下使用

### load

由于调用load方法时的环境很不安全，我们应该尽量减少load方法的逻辑。另一个原因是load方法是线程安全的，它内部使用了锁，所以我们应该避免线程阻塞在load方法中

load很常见的一个使用场景,交换两个方法的实现

```
//摘自MJRefresh
+ (void)load
{
    [self exchangeInstanceMethod1:@selector(reloadData) method2:@selector(mj_reloadData)];
    [self exchangeInstanceMethod1:@selector(reloadRowsAtIndexPaths:withRowAnimation:) method2:@selector(mj_reloadRowsAtIndexPaths:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(deleteRowsAtIndexPaths:withRowAnimation:) method2:@selector(mj_deleteRowsAtIndexPaths:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(insertRowsAtIndexPaths:withRowAnimation:) method2:@selector(mj_insertRowsAtIndexPaths:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(reloadSections:withRowAnimation:) method2:@selector(mj_reloadSections:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(deleteSections:withRowAnimation:) method2:@selector(mj_deleteSections:withRowAnimation:)];
    [self exchangeInstanceMethod1:@selector(insertSections:withRowAnimation:) method2:@selector(mj_insertSections:withRowAnimation:)];
}

+ (void)exchangeInstanceMethod1:(SEL)method1 method2:(SEL)method2
{
    method_exchangeImplementations(class_getInstanceMethod(self, method1), class_getInstanceMethod(self, method2));
}
```

### initialize 

initialize方法主要用来对一些不方便在编译期初始化的对象进行赋值。比如NSMutableArray这种类型的实例化依赖于runtime的消息发送，所以显然无法在编译器初始化：

```

// In Person.m
// int类型可以在编译期赋值
static int someNumber = 0; 
static NSMutableArray *someArray;
+ (void)initialize {
    if (self == [Person class]) {
        // 不方便编译期复制的对象在这里赋值
        someArray = [[NSMutableArray alloc] init];
    }
}

```

## 总结

### load和initialize的共同点

1.如果父类和子类都被调用,父类的调用一定在子类之前

### load方法要点

当类被引用进项目的时候就会执行load函数(在main函数开始执行之前）,与这个类是否被用到无关,每个类的load函数只会自动调用一次.由于load函数是系统自动加载的，因此不需要再调用[super load]，否则父类的load函数会多次执行。

1.当父类和子类都实现load函数时,父类的load方法执行顺序要优先于子类

2.当一个类未实现load方法时,不会调用父类load方法

3.类中的load方法执行顺序要优先于类别(Category)

4.当有多个类别(Category)都实现了load方法,这几个load方法都会执行,但执行顺序不确定(其执行顺序与类别在Compile Sources中出现的顺序一致)

5.当然当有多个不同的类的时候,每个类load 执行顺序与其在Compile Sources出现的顺序一致

注意:
load调用时机比较早,当load调用时,其他类可能还没加载完成,运行环境不安全.
load方法是线程安全的，它使用了锁，我们应该避免线程阻塞在load方法.

### +initialize方法要点

initialize在类或者其子类的第一个方法被调用前调用。即使类文件被引用进项目,但是没有使用,initialize不会被调用。由于是系统自动调用，也不需要显式的调用父类的initialize，否则父类的initialize会被多次执行。假如这个类放到代码中，而这段代码并没有被执行，这个函数是不会被执行的。

1.父类的initialize方法会比子类先执行

2.当子类不实现initialize方法，会把父类的实现继承过来调用一遍。在此之前，父类的方法会被优先调用一次

3.当有多个Category都实现了initialize方法,会覆盖类中的方法,只执行一个(会执行Compile Sources 列表中最后一个Category 的initialize方法)

注意:
在initialize方法收到调用时,运行环境基本健全。
initialize内部也使用了锁，所以是线程安全的。但同时要避免阻塞线程，不要再使用锁



