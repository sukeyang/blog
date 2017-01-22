---
title: iOS运行时更改App语言
date: 2017-01-22 23:25:42
tags: iOS
categories: 技术相关
---

## 需求

主要是方便研发与测试同学的使用，便于验证App在不同语言下的显示效果，希望能够在现有工程不改变大量代码的情况下，尽量满足下面的需求

- 不重启App
- 不改变调用方式（仍然用`NSLocalizedString`）
- 可设置默认语言
- 缺失语言提醒/警告
- 仅在`DEBUG`环境生效，不能影响生产环境
- 对webview等系统组件也能够生效

## 分析

多语言文件会被存放在`<App Bundle Path>/<Language Id>.lproj/`下，其中`*.string`在工程中是`"key" = "value";`的文本形式，编译后实际上是二进制的plist文件，可以用`plutil -p /path/to/xxx.strings`查看。

关于语言和地区ID：
[Language and Locale IDs - Apple Developer](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPInternational/LanguageandLocaleIDs/LanguageandLocaleIDs.html)

项目里用到webview的地方很多，访问不同的页面时也需要支持`Accept-Language`的切换。

虽然是个蛮小的需求，但是网上找的些方法暂时还满足不了（也有可能我搜索姿势不太对）

### 现有方案

#### 1、更改 `NSUserDefaults`中`AppleLanguages`字段的值

这是比较被推荐的办法，缺点是重启应用后才生效

```
[[NSUserDefaults standardUserDefaults] setObject:[NSArray arrayWithObjects:@"en", nil] forKey:@"AppleLanguages"];
[[NSUserDefaults standardUserDefaults] synchronize];
```

#### 2、替换`NSLocalizedString`宏

首先实现一个运行时获取多语言文案的方法，比如：

```
@implementation NSBundle (RunTimeLanguage)

+ (NSString *)runTimeLocalizedStringForKey:(NSString *)key {
	// TODO
	NSString *languageId = @"en";
	
	NSString *path = [[NSBundle mainBundle] pathForResource: languageId ofType:@"lproj"];
    NSBundle *languageBundle = [NSBundle bundleWithPath:path];
    return [languageBundle localizedStringForKey:key value:key table:nil];
}

@end
```

然后在`PrefixHeader.pch`中添加：

```
#undef NSLocalizedString
#define NSLocalizedString(key, comment) [NSBundle runTimeLocalizedStringForKey:key]
```

缺点是需要在预编译头文件里添加内容，感觉不太优雅。还有这样对webview不生效。

### TODO

方法一使用的是系统的选取多语言方法，不重启应用切换语言的功能看似无法支持。
方法二的实现会导致类似webview的组件仍然使用了系统语言。

感觉两个方法有点冲突- -ll

争取在假期里完成这个三方库

（有人说todo就是再也不do，希望不是这个样子😊）

不管怎样，去年的指标在年前算是补上了：）@桃小七
