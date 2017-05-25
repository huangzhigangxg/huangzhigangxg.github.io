---
layout: post

title: 理解 Objective-C：Category 

date: 2017-03-16 15:32:24.000000000 +09:00

---

## 想了解Category如何在运行时为OC的类添加方法的,根据Category实现原理，进一步确定Category的方法为什么会覆盖Class自身类的实现？
> TODO:Swift的 class Extension，protocol Extension 又是如何给类添加方法的，当方法发生覆盖时，怎么确定哪一个方法被调用？函数的派发方式？

```
#import "MyClass.h"

@implementation MyClass

- (void)printName
{
    NSLog(@"%@",@"MyClass");
}

@end

@implementation MyClass(MyAddition)

- (void)printName
{
    NSLog(@"%@",@"MyAddition");
}

@end

```

## 经过 `clang -rewrite-objc MyClass.m` 命令看看Category底层结构体具体实现。

> 通过 clang -rewrite-objc 命令，仅将扩展语法通过可读性更高的 C 语法进行改写，而不是编译期中的子编译过程

```c++
// @implementation MyClass

// - (void)printName {   NSLog(@"%@",@"MyClass"); } 被编译成一个静态函数叫 _I_MyClass_printName 这个名字时编译器决定的.参数时传入 typedef struct obj_object MyClass 也就是 self指针，和 _cmd 调用NSLog()方法，参数就是这两个被定义在常量区的字符串。

static void _I_MyClass_printName(MyClass ` self, SEL _cmd) {
    NSLog((NSString `)&__NSConstantStringImpl__var_folders_tt_zt3fc7zx0xj2fnbc6r1l01n00000gn_T_main_ff014a_mi_0,(NSString `)&__NSConstantStringImpl__var_folders_tt_zt3fc7zx0xj2fnbc6r1l01n00000gn_T_main_ff014a_mi_1);
}

// @end

// @implementation MyClass(MyAddition)

// - (void)printName {   NSLog(@"%@",@"MyAddition"); }

static void _I_MyClass_MyAddition_printName(MyClass ` self, SEL _cmd) {
    NSLog((NSString `)&__NSConstantStringImpl__var_folders_tt_zt3fc7zx0xj2fnbc6r1l01n00000gn_T_main_ff014a_mi_2,(NSString `)&__NSConstantStringImpl__var_folders_tt_zt3fc7zx0xj2fnbc6r1l01n00000gn_T_main_ff014a_mi_3);
}
//编译器生成了实例方法列表OBJC$_CATEGORY_INSTANCE_METHODSMyClass$_MyAddition
//和属性列表OBJC$_PROP_LISTMyClass$_MyAddition，
//两者的命名都遵循了公共前缀+类名+category名字的命名方式


// @end

// 下面这些基础的结构体:_prop_t,_objc_method,_protocol_t,_ivar_t,_class_ro_t,_category_t....将用在编译器处理MyClass和MyAddition上。
struct _prop_t { //属性的结构体
	const char `name;
	const char `attributes;
};

struct _protocol_t;

struct _objc_method { //Method
	struct objc_selector ` _cmd; //SEL
	const char `method_type;
	void  `_imp;//IMP
};

struct _protocol_t {
	void ` isa;  // NULL
	const char `protocol_name;
	const struct _protocol_list_t ` protocol_list; // super protocols
	const struct method_list_t `instance_methods;
	const struct method_list_t `class_methods;
	const struct method_list_t `optionalInstanceMethods;
	const struct method_list_t `optionalClassMethods;
	const struct _prop_list_t ` properties;
	const unsigned int size;  // sizeof(struct _protocol_t)
	const unsigned int flags;  // = 0
	const char `` extendedMethodTypes;
};

struct _ivar_t {  //成员变量结构体
	unsigned long int `offset;  // pointer to ivar offset location
	const char `name;
	const char `type;
	unsigned int alignment;
	unsigned int  size;
};

struct _class_ro_t { // 类里面只读部分的结构
	unsigned int flags;
	unsigned int instanceStart;
	unsigned int instanceSize;
	unsigned int reserved;
	const unsigned char `ivarLayout;
	const char `name;
	const struct _method_list_t `baseMethods;
	const struct _objc_protocol_list `baseProtocols;
	const struct _ivar_list_t `ivars;
	const unsigned char `weakIvarLayout;
	const struct _prop_list_t `properties;
};

struct _class_t { //表达类结构
	struct _class_t `isa;
	struct _class_t `superclass;
	void `cache; //缓存列表
	void `vtable; //这还有个虚函数表
	struct _class_ro_t `ro; //只读的固定信息
};

struct _category_t { //category结构体
	const char `name;
	struct _class_t `cls;
	const struct _method_list_t `instance_methods;
	const struct _method_list_t `class_methods;
	const struct _protocol_list_t `protocols;
	const struct _prop_list_t `properties;
};
extern "C" __declspec(dllimport) struct objc_cache _objc_empty_cache;
#pragma warning(disable:4273)

static struct /`_method_list_t`/ {
	unsigned int entsize;  // sizeof(struct _objc_method)
	unsigned int method_count;
	struct _objc_method method_list[1];
} _OBJC_$_INSTANCE_METHODS_MyClass __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	sizeof(_objc_method),
	1,
	{{(struct objc_selector `)"printName", "v16@0:8", (void `)_I_MyClass_printName}}
}; // 用于MyClass实例方法的结构体列表：_OBJC_$_INSTANCE_METHODS_MyClass 在指定section的存放 。

static struct _class_ro_t _OBJC_METACLASS_RO_$_MyClass __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	1, sizeof(struct _class_t), sizeof(struct _class_t), 
	(unsigned int)0, 
	0, 
	"MyClass",
	0, 
	0, 
	0, 
	0, 
	0, 
};//生成MyClass元类只读信息部分的结构体

static struct _class_ro_t _OBJC_CLASS_RO_$_MyClass __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	0, sizeof(struct MyClass_IMPL), sizeof(struct MyClass_IMPL), 
	(unsigned int)0, 
	0, 
	"MyClass",
	(const struct _method_list_t `)&_OBJC_$_INSTANCE_METHODS_MyClass,
	0, 
	0, 
	0, 
	0, 
};//生成MyClass类的结构体

extern "C" __declspec(dllimport) struct _class_t OBJC_METACLASS_$_NSObject;

extern "C" __declspec(dllexport) struct _class_t OBJC_METACLASS_$_MyClass __attribute__ ((used, section ("__DATA,__objc_data"))) = {
	0, // &OBJC_METACLASS_$_NSObject,
	0, // &OBJC_METACLASS_$_NSObject,
	0, // (void `)&_objc_empty_cache,
	0, // unused, was (void `)&_objc_empty_vtable,
	&_OBJC_METACLASS_RO_$_MyClass,
}; //生成MyClass的元类的结构体。

extern "C" __declspec(dllimport) struct _class_t OBJC_CLASS_$_NSObject;

extern "C" __declspec(dllexport) struct _class_t OBJC_CLASS_$_MyClass __attribute__ ((used, section ("__DATA,__objc_data"))) = {
	0, // &OBJC_METACLASS_$_MyClass,
	0, // &OBJC_CLASS_$_NSObject,
	0, // (void `)&_objc_empty_cache,
	0, // unused, was (void `)&_objc_empty_vtable,
	&_OBJC_CLASS_RO_$_MyClass,
};//生产MyClass的类结构体


static void OBJC_CLASS_SETUP_$_MyClass(void ) {
	OBJC_METACLASS_$_MyClass.isa = &OBJC_METACLASS_$_NSObject;
	OBJC_METACLASS_$_MyClass.superclass = &OBJC_METACLASS_$_NSObject;
	OBJC_METACLASS_$_MyClass.cache = &_objc_empty_cache;
	OBJC_CLASS_$_MyClass.isa = &OBJC_METACLASS_$_MyClass;
	OBJC_CLASS_$_MyClass.superclass = &OBJC_CLASS_$_NSObject;
	OBJC_CLASS_$_MyClass.cache = &_objc_empty_cache;
}//在内存中拼接MyClass所有信息的接口，用于之后统一所有类载入内存用

#pragma section(".objc_inithooks$B", long, read, write)
__declspec(allocate(".objc_inithooks$B")) static void `OBJC_CLASS_SETUP[] = {
	(void `)&OBJC_CLASS_SETUP_$_MyClass,
};//在内存中统一设置所有Objc类结构。

static struct /`_method_list_t`/ {
	unsigned int entsize;  // sizeof(struct _objc_method)
	unsigned int method_count;
	struct _objc_method method_list[1];
} _OBJC_$_CATEGORY_INSTANCE_METHODS_MyClass_$_MyAddition __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	sizeof(_objc_method),
	1,
	{{(struct objc_selector `)"printName", "v16@0:8", (void `)_I_MyClass_MyAddition_printName}}
};//生成MyClass(MyAddition)的Category的方法列表结构体，用于生成category_t

static struct /`_prop_list_t`/ {
	unsigned int entsize;  // sizeof(struct _prop_t)
	unsigned int count_of_properties;
	struct _prop_t prop_list[1];
} _OBJC_$_PROP_LIST_MyClass_$_MyAddition __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	sizeof(_prop_t),
	1,
	{{"name","T@\"NSString\",C,N"}}
};//生产MyClass(MyAddition)的Category的属性列表结构体，用于生成category_t

extern "C" __declspec(dllexport) struct _class_t OBJC_CLASS_$_MyClass;

static struct _category_t _OBJC_$_CATEGORY_MyClass_$_MyAddition __attribute__ ((used, section ("__DATA,__objc_const"))) = 
{
	"MyClass",
	0, // &OBJC_CLASS_$_MyClass,
	(const struct _method_list_t `)&_OBJC_$_CATEGORY_INSTANCE_METHODS_MyClass_$_MyAddition,
	0,
	0,
	(const struct _prop_list_t `)&_OBJC_$_PROP_LIST_MyClass_$_MyAddition,
};//最后拼接成MyClass(MyAddition)的Category的完整结构。
static void OBJC_CATEGORY_SETUP_$_MyClass_$_MyAddition(void ) {
	_OBJC_$_CATEGORY_MyClass_$_MyAddition.cls = &OBJC_CLASS_$_MyClass;
}//为Category(MyAddition)设置扩展源类，拿到原类结构体后指针应该会对原类有修改eg：插入扩展方法。
#pragma section(".objc_inithooks$B", long, read, write)
__declspec(allocate(".objc_inithooks$B")) static void `OBJC_CATEGORY_SETUP[] = {
	(void `)&OBJC_CATEGORY_SETUP_$_MyClass_$_MyAddition,
};
static struct _class_t `L_OBJC_LABEL_CLASS_$ [1] __attribute__((used, section ("__DATA, __objc_classlist,regular,no_dead_strip")))= {
	&OBJC_CLASS_$_MyClass,
};
static struct _category_t `L_OBJC_LABEL_CATEGORY_$ [1] __attribute__((used, section ("__DATA, __objc_catlist,regular,no_dead_strip")))= {
	&_OBJC_$_CATEGORY_MyClass_$_MyAddition,
};
static struct IMAGE_INFO { unsigned version; unsigned flag; } _OBJC_IMAGE_INFO = { 0, 2 };

```
## 上面的MyClass类和它的Category经过Clang编译器。翻译成了C的各种结构体与函数，将它们拼接起来在内存中，表达完整的OC类内存模型。
## 之后看看这些生成的category结构体 如何通过运行时代码实现扩展类功能的办法。

## 下面是运行时代码入口，Category会一步步将自己的方法插入Class的class_rw_t的methods里面。重复的key就会覆盖。

```
void _objc_init(void)
└──const char `map_2_images(...)
    └──const char `map_images_nolock(...)
        └──void _read_images(header_info ``hList, uint32_t hCount)
```
## 下面是添加category给class片段 

```
if (cat->classMethods  ||  cat->protocols  
    /` ||  cat->classProperties `/) {
    addUnattachedCategoryForClass(cat, cls->ISA(), hi);
    if (cls->ISA()->isRealized()) {
        remethodizeClass(cls->ISA());
    }
}
```
## `addUnattachedCategoryForClass` 函数语意上就是添加还没有被添加的Category给Class。
## `remethodizeClass(cls->ISA());` 重新调整class的methods结构体。


```
static void addUnattachedCategoryForClass(category_t `cat, Class cls,
                                          header_info `catHeader)
{
    runtimeLock.assertWriting();

    NXMapTable `cats = unattachedCategories(); //这里获得一张巨大的静态表，里面有所有还没添加到class上的Category
    category_list `list;

    list = (category_list `)NXMapGet(cats, cls); //以cls为key在表中查找属于这个class类的Category的方法列表list。
    if (!list) {
        list = (category_list `)
            calloc(sizeof(`list) + sizeof(list->list[0]), 1);
    } else {
        list = (category_list `)
            realloc(list, sizeof(`list) + sizeof(list->list[0]) ` (list->count + 1));
    }
    list->list[list->count++] = (locstamped_category_t){cat, catHeader};
    NXMapInsert(cats, cls, list); //将这个获得的方法列表list插入这张大表里 以cls为key，list为value。
}

static void remethodizeClass(Class cls)
{
		//....///
    if ((cats = unattachedCategoriesForClass(cls, false/`not realizing`/))) {
		//....///
        attachCategories(cls, cats, true /`flush caches`/);        
        free(cats);
    }
}

```
## `attachCategories`里就会将方法列表重新写到cls->data()-> methods里面。

```
static void 
attachCategories(Class cls, category_list `cats, bool flush_caches)
{
    method_list_t ``mlists = (method_list_t ``)
        malloc(cats->count ` sizeof(`mlists));
    while (i--) {
        method_list_t `mlist = entry.cat->methodsForMeta(isMeta);
        if (mlist) {
            mlists[mcount++] = mlist;
            fromBundle |= entry.hi->isBundle();
        }
    } //mlists 为我们要的方法列表指针

    auto rw = cls->data();

    prepareMethodLists(cls, mlists, mcount, NO, fromBundle);
    rw->methods.attachLists(mlists, mcount);//添加mlists方法到cls-> data()->methods里
    free(mlists);
    if (flush_caches  &&  mcount > 0) flushCaches(cls);

    rw->properties.attachLists(proplists, propcount);
    free(proplists);

    rw->protocols.attachLists(protolists, protocount);
    free(protolists);
}

```

+ [objc_msgSend消息传递](http://www.cnblogs.com/fengmin/p/5820453.html)
+ [深入理解Objective-C：Category](http://tech.meituan.com/DiveIntoCategory.html)
