## 中文说明
#### `LSSafeProtector` 是一个可快速集成但功能强大的防止crash库,使用Objective-C编写.


## 须知
LSSafeProtector 基于 "Xcode 7.3 , iOS 6+ 和ARC ，请使用最新正式版来编译LSSafeProtector,旧版本的Xcode可能有效，但不保证会出现一些兼容性问题。


## CocoaPods

推荐使用 CocoaPods 安装。

1. 在 Podfile 中添加 `pod 'LSSafeProtector'`。
2. 执行 `pod install` 或 `pod update`。(如搜索不到,请更新pod仓库pod repo update)
3. 导入 `"LSSafeProtector.h"`。

## 手动安装
通过 Clone or download 下载 LSSafeProtector 文件夹内的所有内容。
将 LSSafeProtector 内的源文件添加(拖放)到你的工程。
导入 LSSafeProtector.h 。
## 使用

- 通过如下方式开启防止闪退功能,debug模式会打印crash日志，同时会利用断言来让程序闪退，也会回调block,达到测试环境及时发现及时修改，Release模式既不打印也不会断言闪退，会回调block，自己可以上传exception到bugly(注意线上环境isDebug一定要设置为NO)

```
//注意线上环境isDebug一定要设置为NO)
[LSSafeProtector openSafeProtectorWithIsDebug:YES block:^(NSException *exception, LSSafeProtectorCrashType crashType) {
//[Bugly reportException:exception];

//此方法相对于上面的方法，好处在于bugly后台查看bug崩溃位置时，不用点击跟踪数据，再点击crash_attach.log，查看里面的额外信息来查看崩溃位置
[Bugly reportExceptionWithCategory:3 name:exception.name reason:[NSString stringWithFormat:@"%@  崩溃位置:%@",exception.reason,exception.userInfo[@"location"]] callStack:@[exception.userInfo[@"callStackSymbols"]] extraInfo:exception.userInfo terminateApp:NO];
}];
```
更多的使用用例可以看Demo工程演示


### 下面是防止崩溃的效果

- 可导致崩溃的代码
```
NSMutableArray *a1=[NSMutableArray array];
a1[10];
```
- 若没有防止崩溃，则会直接崩溃，如下图所示
![image](https://github.com/lsmakethebest/LSSafeProtector/blob/master/images/1.png)
- 用本框架来防止崩溃，则会捕获到崩溃信息并打印出来(测试环境会利用断言闪退达到及时发现及时修改)，如下图
![image](https://github.com/lsmakethebest/LSSafeProtector/blob/master/images/2.png)
- 来看看block回调回来的信息都有哪些
![image](https://github.com/lsmakethebest/LSSafeProtector/blob/master/images/3.png)
- KVO  检测到dealloc时有没remove的keyPath
![image](https://github.com/lsmakethebest/LSSafeProtector/blob/master/images/4.png)

### 
### 目前支持以下类型crash
-  1、LSSafeProtectorCrashTypeSelector
```
1.捕获到未实现方法时，自动将消息转发，避免crash
```
-  2、LSSafeProtectorCrashTypeKVO
```
1.移除未注册的观察者 会crash
2.重复移除观察者 会crash
3.添加了观察者但没有实现observeValueForKeyPath:ofObject:change:context:方法
4.添加移除keypath=nil;
5.添加移除observer=nil;
6.dealloc时自动移除观察者，俗称自释放KVO
```
- 3、LSSafeProtectorCrashTypeNSArray
```
1. NSArray的快速创建方式 NSArray *array = @[@"chenfanfang", @"AvoidCrash"];//调用的是3的方法
2. + (instancetype)arrayWithObjects:(const ObjectType _Nonnull [_Nonnull])objects count:(NSUInteger)cnt;调用的也是3的方法
3. - (instancetype)initWithObjects:(const ObjectType _Nonnull [_Nullable])objects count
4. - (id)objectAtIndex:(NSUInteger)index
```

- 4、LSSafeProtectorCrashTypeNSMutableArray
```
1. - (void)addObject:(ObjectType)anObject(实际调用insertObject:)
2. - (void)insertObject:(ObjectType)anObject atIndex:(NSUInteger)index;
3. - (id)objectAtIndex:(NSUInteger)index( 包含   array[index] 形式)
4. - (void)removeObjectAtIndex:(NSUInteger)index
5. - (void)replaceObjectAtIndex:(NSUInteger)index
```
- 5、LSSafeProtectorCrashTypeNSDictionary
```
1.+ (instancetype)dictionaryWithObjects:(const ObjectType _Nonnull [_Nullable])objects forKeys:(const KeyType <NSCopying> _Nonnull [_Nullable])keys count:(NSUInteger)cnt会调用2中的方法
2.- (instancetype)initWithObjects:(const ObjectType _Nonnull [_Nullable])objects forKeys:(const KeyType _Nonnull [_Nullable])keys count:(NSUInteger)cnt;
3. @{@"key1":@"value1",@"key2":@"value2"}也会调用2中的方法
4. - (instancetype)initWithObjects:(NSArray<ObjectType> *)objects forKeys:(NSArray<KeyType <NSCopying>> *)keys;
```
- 6、LSSafeProtectorCrashTypeNSMutableDictionary
```
1.直接调用 setObject:forKey
2.通过下标方式赋值的时候，value为nil不会崩溃
iOS11之前会调用 setObject:forKey
iOS11之后（含11)  setObject:forKeyedSubscript:
3.removeObjectForKey
```
- 7、LSSafeProtectorCrashTypeNSStirng
```
1. initWithString
2. hasPrefix
3. hasSuffix
4. substringFromIndex:(NSUInteger)from
5. substringToIndex:(NSUInteger)to {
6. substringWithRange:(NSRange)range {
7. characterAtIndex:(NSUInteger)index
8. stringByReplacingOccurrencesOfString:(NSString *)target withString:(NSString *)replacement 实际上调用的是9方法
9. stringByReplacingOccurrencesOfString:(NSString *)target withString:(NSString *)replacement options:(NSStringCompareOptions)options range:(NSRange)searchRange
10. stringByReplacingCharactersInRange:(NSRange)range withString:(NSString *)replacement
```

- 8、LSSafeProtectorCrashTypeNSMutableString
```
//除NSString的一些方法外又额外避免了一些方法crash
1.- (void)replaceCharactersInRange:(NSRange)range withString:(NSString *)aString;
2.- (NSUInteger)replaceOccurrencesOfString:(NSString *)target withString:(NSString *)replacement options:(NSStringCompareOptions)options range:(NSRange)searchRange;
3.- (void)insertString:(NSString *)aString atIndex:(NSUInteger)loc;
4.- (void)deleteCharactersInRange:(NSRange)range;
5.- (void)appendString:(NSString *)aString;
6.- (void)setString:(NSString *)aString;
```
- 9、LSSafeProtectorCrashTypeNSAttributedString
```
1.- (instancetype)initWithString:(NSString *)str;
2.- (instancetype)initWithString:(NSString *)str attributes:(nullable NSDictionary<NSAttributedStringKey, id> *)attrs;
3.- (instancetype)initWithAttributedString:(NSAttributedString *)attrStr;
```
- 10、LSSafeProtectorCrashTypeNSMutableAttributedString
```
1.- (instancetype)initWithString:(NSString *)str;
2.- (instancetype)initWithString:(NSString *)str attributes:(nullable NSDictionary<NSAttributedStringKey, id> *)attrs;
3.- (instancetype)initWithAttributedString:(NSAttributedString *)attrStr;

4. - (void)replaceCharactersInRange:(NSRange)range withString:(NSString *)str;
5.- (void)setAttributes:(nullable NSDictionary<NSAttributedStringKey, id> *)attrs range:(NSRange)range;

6.- (void)addAttribute:(NSAttributedStringKey)name value:(id)value range:(NSRange)range;
7.- (void)addAttributes:(NSDictionary<NSAttributedStringKey, id> *)attrs range:(NSRange)range;
8.- (void)removeAttribute:(NSAttributedStringKey)name range:(NSRange)range;

9.- (void)replaceCharactersInRange:(NSRange)range withAttributedString:(NSAttributedString *)attrString;
10.- (void)insertAttributedString:(NSAttributedString *)attrString atIndex:(NSUInteger)loc;
11.- (void)appendAttributedString:(NSAttributedString *)attrString;
12.- (void)deleteCharactersInRange:(NSRange)range;
13.- (void)setAttributedString:(NSAttributedString *)attrString;

```
- 11、LSSafeProtectorCrashTypeNSNotificationCenter
```
1. dealloc时自动将self从通知中心移除

```
# 联系    
- 如果使用过程中遇到什么问题或有什么建议可以联系我，我会在收到后尽快回复您
- email at: song@ysui.cn

- QQ at:623501561 


## 许可
LSSafeProtector 使用 MIT 许可证，详情可见 [LICENSE](LICENSE) 文件。



