---
layout: post
title: "获取一个NSDictionary，装有自身的所有属性和值"
date: 2013-09-10 14:27
comments: true
categories: iOS
---

- NSObject , 获取NSObject的所有属性
近期做项目需要用到一个能返回自身所有属性的方法
查了下资料发现objc/runtime.h可以实现
不过必须是使用@synthesize定义的属性
具体的代码如下：

{% codeblock lang:objc %}

- (NSDictionary *)properties{
    unsigned int outCount,index;
    objc_property_t* properties_t = class_copyPropertyList([self class], &outCount);
    NSMutableDictionary *resultDictionary = [[NSMutableDictionary alloc] init];
    for(index = 0 ; index < outCount ; index ++){
        objc_property_t t_property = properties_t[index];
        NSString *attributeName = [NSString stringWithUTF8String:property_getName(t_property)];
        if(class_respondsToSelector([self class], NSSelectorFromString(attributeName))){
            [resultDictionary setObject:[self valueForKey:attributeName]
                                 forKey:attributeName];
        }
    }
    NSDictionary *res = [NSDictionary dictionaryWithDictionary:resultDictionary];
    [resultDictionary release];
    return res;
}

- (void)setDataWithDictionary:(NSDictionary *)dataDictionary{
    
    [dataDictionary enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
        if(class_respondsToSelector([self class], NSSelectorFromString(key))){
            [self setValue:obj forKey:key];
        }else{
            NSLog(@"Property:%@ can't set the value",key);
        }
    }];
}

{% endcodeblock %}