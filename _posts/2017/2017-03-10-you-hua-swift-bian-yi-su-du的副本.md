---
layout: post

title: 理解 Objective-C：Category 

date: 2017-03-16 15:32:24.000000000 +09:00

---

想了解Category如何在运行时为OC的类添加方法的

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

经过 `clang -rewrite-objc MyClass.m` 命令看看Category底层结构体具体实现。

```C++
// @implementation MyClass


static void _I_MyClass_printName(MyClass * self, SEL _cmd) {
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_tt_zt3fc7zx0xj2fnbc6r1l01n00000gn_T_main_ff014a_mi_0,(NSString *)&__NSConstantStringImpl__var_folders_tt_zt3fc7zx0xj2fnbc6r1l01n00000gn_T_main_ff014a_mi_1);
}

// @end

// @implementation MyClass(MyAddition)


static void _I_MyClass_MyAddition_printName(MyClass * self, SEL _cmd) {
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_tt_zt3fc7zx0xj2fnbc6r1l01n00000gn_T_main_ff014a_mi_2,(NSString *)&__NSConstantStringImpl__var_folders_tt_zt3fc7zx0xj2fnbc6r1l01n00000gn_T_main_ff014a_mi_3);
}

// @end

struct _prop_t {
	const char *name;
	const char *attributes;
};

struct _protocol_t;

struct _objc_method {
	struct objc_selector * _cmd;
	const char *method_type;
	void  *_imp;
};

struct _protocol_t {
	void * isa;  // NULL
	const char *protocol_name;
	const struct _protocol_list_t * protocol_list; // super protocols
	const struct method_list_t *instance_methods;
	const struct method_list_t *class_methods;
	const struct method_list_t *optionalInstanceMethods;
	const struct method_list_t *optionalClassMethods;
	const struct _prop_list_t * properties;
	const unsigned int size;  // sizeof(struct _protocol_t)
	const unsigned int flags;  // = 0
	const char ** extendedMethodTypes;
};

struct _ivar_t {
	unsigned long int *offset;  // pointer to ivar offset location
	const char *name;
	const char *type;
	unsigned int alignment;
	unsigned int  size;
};

struct _class_ro_t {
	unsigned int flags;
	unsigned int instanceStart;
	unsigned int instanceSize;
	unsigned int reserved;
	const unsigned char *ivarLayout;
	const char *name;
	const struct _method_list_t *baseMethods;
	const struct _objc_protocol_list *baseProtocols;
	const struct _ivar_list_t *ivars;
	const unsigned char *weakIvarLayout;
	const struct _prop_list_t *properties;
};

struct _class_t {
	struct _class_t *isa;
	struct _class_t *superclass;
	void *cache;
	void *vtable;
	struct _class_ro_t *ro;
};

struct _category_t {
	const char *name;
	struct _class_t *cls;
	const struct _method_list_t *instance_methods;
	const struct _method_list_t *class_methods;
	const struct _protocol_list_t *protocols;
	const struct _prop_list_t *properties;
};
extern "C" __declspec(dllimport) struct objc_cache _objc_empty_cache;
#pragma warning(disable:4273)

static struct /*_method_list_t*/ {
	unsigned int entsize;  // sizeof(struct _objc_method)
	unsigned int method_count;
	struct _objc_method method_list[1];
} _OBJC_$_INSTANCE_METHODS_MyClass __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	sizeof(_objc_method),
	1,
	{{(struct objc_selector *)"printName", "v16@0:8", (void *)_I_MyClass_printName}}
};

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
};

static struct _class_ro_t _OBJC_CLASS_RO_$_MyClass __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	0, sizeof(struct MyClass_IMPL), sizeof(struct MyClass_IMPL), 
	(unsigned int)0, 
	0, 
	"MyClass",
	(const struct _method_list_t *)&_OBJC_$_INSTANCE_METHODS_MyClass,
	0, 
	0, 
	0, 
	0, 
};

extern "C" __declspec(dllimport) struct _class_t OBJC_METACLASS_$_NSObject;

extern "C" __declspec(dllexport) struct _class_t OBJC_METACLASS_$_MyClass __attribute__ ((used, section ("__DATA,__objc_data"))) = {
	0, // &OBJC_METACLASS_$_NSObject,
	0, // &OBJC_METACLASS_$_NSObject,
	0, // (void *)&_objc_empty_cache,
	0, // unused, was (void *)&_objc_empty_vtable,
	&_OBJC_METACLASS_RO_$_MyClass,
};

extern "C" __declspec(dllimport) struct _class_t OBJC_CLASS_$_NSObject;

extern "C" __declspec(dllexport) struct _class_t OBJC_CLASS_$_MyClass __attribute__ ((used, section ("__DATA,__objc_data"))) = {
	0, // &OBJC_METACLASS_$_MyClass,
	0, // &OBJC_CLASS_$_NSObject,
	0, // (void *)&_objc_empty_cache,
	0, // unused, was (void *)&_objc_empty_vtable,
	&_OBJC_CLASS_RO_$_MyClass,
};
static void OBJC_CLASS_SETUP_$_MyClass(void ) {
	OBJC_METACLASS_$_MyClass.isa = &OBJC_METACLASS_$_NSObject;
	OBJC_METACLASS_$_MyClass.superclass = &OBJC_METACLASS_$_NSObject;
	OBJC_METACLASS_$_MyClass.cache = &_objc_empty_cache;
	OBJC_CLASS_$_MyClass.isa = &OBJC_METACLASS_$_MyClass;
	OBJC_CLASS_$_MyClass.superclass = &OBJC_CLASS_$_NSObject;
	OBJC_CLASS_$_MyClass.cache = &_objc_empty_cache;
}
#pragma section(".objc_inithooks$B", long, read, write)
__declspec(allocate(".objc_inithooks$B")) static void *OBJC_CLASS_SETUP[] = {
	(void *)&OBJC_CLASS_SETUP_$_MyClass,
};

static struct /*_method_list_t*/ {
	unsigned int entsize;  // sizeof(struct _objc_method)
	unsigned int method_count;
	struct _objc_method method_list[1];
} _OBJC_$_CATEGORY_INSTANCE_METHODS_MyClass_$_MyAddition __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	sizeof(_objc_method),
	1,
	{{(struct objc_selector *)"printName", "v16@0:8", (void *)_I_MyClass_MyAddition_printName}}
};

static struct /*_prop_list_t*/ {
	unsigned int entsize;  // sizeof(struct _prop_t)
	unsigned int count_of_properties;
	struct _prop_t prop_list[1];
} _OBJC_$_PROP_LIST_MyClass_$_MyAddition __attribute__ ((used, section ("__DATA,__objc_const"))) = {
	sizeof(_prop_t),
	1,
	{{"name","T@\"NSString\",C,N"}}
};

extern "C" __declspec(dllexport) struct _class_t OBJC_CLASS_$_MyClass;

static struct _category_t _OBJC_$_CATEGORY_MyClass_$_MyAddition __attribute__ ((used, section ("__DATA,__objc_const"))) = 
{
	"MyClass",
	0, // &OBJC_CLASS_$_MyClass,
	(const struct _method_list_t *)&_OBJC_$_CATEGORY_INSTANCE_METHODS_MyClass_$_MyAddition,
	0,
	0,
	(const struct _prop_list_t *)&_OBJC_$_PROP_LIST_MyClass_$_MyAddition,
};
static void OBJC_CATEGORY_SETUP_$_MyClass_$_MyAddition(void ) {
	_OBJC_$_CATEGORY_MyClass_$_MyAddition.cls = &OBJC_CLASS_$_MyClass;
}
#pragma section(".objc_inithooks$B", long, read, write)
__declspec(allocate(".objc_inithooks$B")) static void *OBJC_CATEGORY_SETUP[] = {
	(void *)&OBJC_CATEGORY_SETUP_$_MyClass_$_MyAddition,
};
static struct _class_t *L_OBJC_LABEL_CLASS_$ [1] __attribute__((used, section ("__DATA, __objc_classlist,regular,no_dead_strip")))= {
	&OBJC_CLASS_$_MyClass,
};
static struct _category_t *L_OBJC_LABEL_CATEGORY_$ [1] __attribute__((used, section ("__DATA, __objc_catlist,regular,no_dead_strip")))= {
	&_OBJC_$_CATEGORY_MyClass_$_MyAddition,
};

```