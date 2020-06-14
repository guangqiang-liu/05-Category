# 05-Category 本质

category的实现原理？，category的底层数据结构？

我们创建一个`Person`类，然后创建一个`Person+Eat`的分类，代码如下：

`Person类`

```
@interface Person : NSObject

- (void)run;
@end

@implementation Person

- (void)run {
    NSLog(@"---%s", __func__);
}
@end
```

`Person+Eat分类`

```
@interface Person (Eat)

- (void)eat;
@end

@implementation Person (Eat)

- (void)eat {
    NSLog(@"--%s", __func__);
}
@end
```

然后我们执行命令`xcrun -sdk iphoneos  clang  -arch  arm64  -rewrite-objc Person+Eat.m`，将`Person+Eat.m`转换为底层c++代码

从转换后的底层c++代码中，我们可以看到分类的数据结构，结构体`_category_t`如下：

```
struct _category_t {
	const char *name;
	struct _class_t *cls;
	const struct _method_list_t *instance_methods;
	const struct _method_list_t *class_methods;
	const struct _protocol_list_t *protocols;
	const struct _prop_list_t *properties;
};
```

*`_OBJC_$_CATEGORY_Person_$_Eat`：此变量就是给`_category_t`结构体内成员信息进行赋值*

```
static struct _category_t _OBJC_$_CATEGORY_Person_$_Eat __attribute__ ((used, section ("__DATA,__objc_const"))) = 
{
	"Person", // 类名
	0, // &OBJC_CLASS_$_Person, // cls
	(const struct _method_list_t *)&_OBJC_$_CATEGORY_INSTANCE_METHODS_Person_$_Eat, // 实例方法列表
	0, // 类方法列表
	0, // 协议列表
	0, // 属性列表
};
```

我们知道运行时机制会将工程中所有的分类都转换为`_category_t`这种结构体对象，所有分类的信息也都临时存放在`_category_t`结构体中，然后程序会将所有的分类信息都合并到类对象\元类对象的结构体中，这里合并的信息主要包括如下几个：

1. 方法列表
2. 属性列表
3. 协议列表

运行时的具体合并操作底层源码解读如下图所示：

![](https://imgs-1257778377.cos.ap-shanghai.myqcloud.com/QQ20200204-171415@2x.png)

类中的方法和分类中添加的方法的调用顺序?
> 最后编译的分类，优先放到分类方法数组的最前面，合并后在方法列表的前面，在查找方法调用时，也就会优先调用
> 
> OC类文件的编译顺序就是`Compile Sources`的文件的存放顺序

![](https://imgs-1257778377.cos.ap-shanghai.myqcloud.com/QQ20200323-161611@2x.png)

---

分类中的`load`方法有什么作用，调用时机和调用顺序?
> `load`方法会在runtime运行时加载类、分类信息时调用，并且每个类\分类的`load`方法在程序运行过程中只会调用一次

`load`方法的调用顺序如下图所示：

![](https://imgs-1257778377.cos.ap-shanghai.myqcloud.com/QQ20200204-202758@2x.png)

`load`方法的调用机制是根据方法内存地址直接调用，并不是执行`objc_msgSend`进行消息发送

---

分类中的`initialize`方法的作用，调用时机和调用顺序？
> `initialize`方法会在**类**第一次接收到消息时调用

`initialize`方法的调用时机和调用顺序如下图所示：

![](https://imgs-1257778377.cos.ap-shanghai.myqcloud.com/QQ20200204-211057@2x.png)

`initialize`方法的调用机制是通过`objc_msgSend`发送消息，所以调用流程也是遵循`objc_msgSend`

讲解示例Demo地址：[https://github.com/guangqiang-liu/05-CategoryDemo]()


## 更多文章
* ReactNative开源项目OneM(1200+star)：**[https://github.com/guangqiang-liu/OneM](https://github.com/guangqiang-liu/OneM)**：欢迎小伙伴们 **star**
* 简书主页：包含多篇iOS和RN开发相关的技术文章[http://www.jianshu.com/u/023338566ca5](http://www.jianshu.com/u/023338566ca5) 欢迎小伙伴们：**多多关注，点赞**
* ReactNative QQ技术交流群(2000人)：**620792950** 欢迎小伙伴进群交流学习
* iOS QQ技术交流群：**678441305** 欢迎小伙伴进群交流学习