# 用反射机制实现二次开发

LayaNative通过提供反射机制来帮助开发者可以方便的进行二次开发, 下面通过示例来了解一下如何进行二次开发。

## 1. 调用静态函数

使用LayaNative, 可以在JavaScript层调用移动端的原生开发语言(Android下Java, iOS下Objective-C)编写的静态函数。

###    1.1 JavaScript层

JavaScript层的调用方式：在构建发布前，使用LayaAir-IDE在脚本中添加如下代码，发布后会将TS编译为JS：

```typescript
  var os = conchConfig.getOS();
  var bridge;
  var obj = {} as any;
  if (os == "Conch-ios") {
      bridge = PlatformClass.createClass("JSBridge");//创建脚步代理
  }
  else if (os == "Conch-android") {
    //需要完整的类路径，注意与iOS的不同
    bridge = PlatformClass.createClass("demo.JSBridge");//创建脚步代理
  } 

  if (os == "Conch-ios") {
    //iOS注意函数签名，注意与Android的不同
    alert(bridge.call("testString:","hello"));
    alert(bridge.call("testNumber:",256.0));
    alert(bridge.call("testBool:",false));
    obj.value = "Hello OC!";
    bridge.callWithBack(function(value: any) {
      var obj = JSON.parse(value)
      alert(obj.value);
      },"testAsyncCallback:", JSON.stringify(obj));
  }
  else if (os == "Conch-android") {
    alert(bridge.call("testString","hello"));
    alert(bridge.call("testNumber",256.0));
    alert(bridge.call("testBool",false));
    obj.value = "Hello Java!";
    bridge.callWithBack(function(value: any) {
      var obj = JSON.parse(value)
      alert(obj.value);
    },"testAsyncCallback",JSON.stringify(obj));
  } 

```

>添加上述代码时，如果某些native中的方法没有定义，可以这样写：`(window as any).xxxx`，例如，`(window as any).conchConfig.getOS()`。



###     1.2 Android/Java层

发布后，使用Android Studio打开项目，在类`JSBridge`中添加下列函数:

```javascript
    public static String testString(String value) {
        Log.d("JSBridge", "java: " + value);
        return "LayaBox";
    }
    public static double testNumber(double value) {
        Log.d("JSBridge", "java: " + value);
        return 512;
    }
    public static boolean testBool(boolean value) {
        Log.d("JSBridge", "java: " + value);
        return value ? false : true;
    }
    public static void testAsyncCallback(String json) {
        //js thread
        try {
            JSONObject root = new JSONObject(json);
            Log.d("JSBridge", "java: " + root.getString( "value" ));
        } catch (JSONException e) {
            e.printStackTrace();
        }
        m_Handler.post(
                new Runnable() {
                    public void run() {
                        //ui thread update ui
                        JSONObject obj = new JSONObject();
                        try {
                            obj.put("value", "Hello JS!");
                        } catch (JSONException e) {
                            e.printStackTrace();
                        }
                        ExportJavaFunction.CallBackToJS(JSBridge.class,"testAsyncCallback", obj.toString());
                    }
                });
    }
```

###     1.3 iOS/OC层

发布后，使用XCode打开项目，在类`JSBridge`中添加下列函数:

```javascript
    +(NSString*)testString:(NSString*)value
    {
      NSLog(@"OC: %@",value);
      return @"LayaBox";
    }
    +(NSNumber*)testNumber:(NSNumber*)value
    {
      NSLog(@"OC: %@",value);
      return @512;
    }
    +(NSNumber*)testBool:(NSNumber*)value
    {
      NSLog(@"OC: %d",value.boolValue);
      return [NSNumber numberWithBool:value.boolValue ? NO : YES];
    }
    +(void)testAsyncCallback:(NSString*)json
    {
      //js thread
      NSError* error = nil;
      NSData* jsonData = [json dataUsingEncoding:NSUTF8StringEncoding];
      NSDictionary* dict = [NSJSONSerialization JSONObjectWithData:jsonData options:NSJSONReadingMutableContainers error:&error];
      NSLog(@"OC: %@", [dict objectForKey:@"value"]);
      dispatch_async(dispatch_get_main_queue(), ^{
          //ui thread
          NSError* error = nil;
          NSDictionary* dic = [NSDictionary dictionaryWithObject:@"Hello JS!" forKey:@"value"];
          NSData* jsonData = [NSJSONSerialization dataWithJSONObject:dic options:NSJSONWritingPrettyPrinted error:&error];
          NSString* jsonStr = [[NSString alloc] initWithData:jsonData encoding:NSUTF8StringEncoding];
          [[conchRuntime GetIOSConchRuntime] callbackToJSWithClass:self.class methodName:@"testAsyncCallback:" ret:jsonStr];
      });
  }

```
注意：

（1）函数参数只支持布尔、浮点、字符串等基本类型，支持返回值。原生函数运行在脚本线程，更新UI需要转到UI线程，支持异步回调函数。

（2）OC源文件后缀要改成.mm，OC的方法是静态的类方法要用+。




##  2. 平台代码（Android/iOS）主动执行js脚本

iOS/OC执行JS脚本：

```javascript
  [[conchRuntime GetIOSConchRuntime] runJS:@"alert('hello')"];
```

Android/Java执行JS脚本：

```javascript
  ConchJNI.RunJS("alert('hello world')");
```

