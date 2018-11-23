# Xposed 模块创建

1. 在 `Android Studio` 新建一个 `project` , 切换至 `Project` 模式， 在 `app/libs` 文件夹下放入下载的api-jar包( [api-jar包下载地址](https://bintray.com/rovo89/de.robv.android.xposed/download_file?file_path=de%2Frobv%2Fandroid%2Fxposed%2Fapi%2F82%2Fapi-82.jar) )

2. **修改 `AndroidManifest.xml` 文件**

   在 `application` 块中加入如下代码：

   ```xml
   <!-- 是否是xposed模块，xposed根据这个来判断是否是模块 -->
   <meta-data
   	android:name="xposedmodule"
   	android:value="true" />
   <!-- 模块描述，显示在xposed模块列表那里第二行 -->
   <meta-data
   	android:name="xposeddescription"
   	android:value="测试Xposed模块" />
   <!-- 最低xposed版本号(lib文件名可知) -->
   <meta-data
     	android:name="xposedminversion"
       android:value="82" />
   
   ```

3. **修改 `build.gradle` **

   如果是 **`Android Studio 2.X`** ，将 `dependencies` 中的 `compile fileTree(dir: 'libs', include: ['*.jar'])` 

   ```groovy
   dependencies {
       compile fileTree(dir: 'libs', include: ['*.jar'])
       testCompile 'junit:junit:4.12'
       compile 'com.android.support:appcompat-7:24.1.1'
   }
   ```

   替换为 `provided fileTree(dir: 'libs', include: ['*.jar'])` 

   ```groovy
   dependencies {
       provided fileTree(dir: 'libs', include: ['*.jar'])
       testCompile 'junit:junit:4.12'
       compile 'com.android.support:appcompat-7:24.1.1'
   }
   ```

   如果是 **`Android Studio 3.X`** ，将 `dependencies` 中的 `implementation fileTree(dir: 'libs', include: ['*.jar'])` 

   ```groovy
   dependencies {
       implementation fileTree(include: ['*.jar'], dir: 'libs')
       implementation 'com.android.support:appcompat-v7:28.0.0-alpha1'
       implementation 'com.android.support.constraint:constraint-layout:1.1.3'
       testImplementation 'junit:junit:4.12'
       androidTestImplementation 'com.android.support.test:runner:1.0.2'
       androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
       implementation files('libs/api-82.jar') //如果有这行需要把它注释掉或者删掉
   }
   ```

   替换为 `compileOnly fileTree(dir: 'libs', include: ['*.jar'])`

   ```groovy
   dependencies {
       compileOnly fileTree(include: ['*.jar'], dir: 'libs')
       implementation 'com.android.support:appcompat-v7:28.0.0-alpha1'
       implementation 'com.android.support.constraint:constraint-layout:1.1.3'
       testImplementation 'junit:junit:4.12'
       androidTestImplementation 'com.android.support.test:runner:1.0.2'
       androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
       //implementation files('libs/api-82.jar') 
   }
   ```

4. **编写 `Xposed` 模块代码**

   在 `app/src/main/java` 下的 `MainActivity` 或者自己新建一个类中编写 `Xposed` 代码

   ```java
   package com.example.alittle.hooktarget;
   
   
   import de.robv.android.xposed.IXposedHookLoadPackage;
   import de.robv.android.xposed.XC_MethodHook;
   import de.robv.android.xposed.XposedBridge;
   import de.robv.android.xposed.XposedHelpers;
   import de.robv.android.xposed.callbacks.XC_LoadPackage;
   
   public class Main implements IXposedHookLoadPackage {
       // 需要 hook 的 app 包名
       private String app_package_name = "com.example.targethook";
       // 需要 hook 的 app 的类名，需要完整路径
       private String app_class_name = "com.example.targethook.MainActivity";
       // 需要 hook 的 app 的方法名
       private String app_method_name = "onCreate";
       @Override
       public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
           if (!lpparam.packageName.equals(app_package_name)){
               return;
           }
           final Class<?> clazz = XposedHelpers.findClass(app_class_name, lpparam.classLoader);
           XposedBridge.hookAllMethods(clazz, app_method_name, new XC_MethodHook() {
               @Override
               protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                   XposedBridge.log("mylog_before");
               }
               @Override
               protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                   XposedBridge.log("mylog_after");
               }
           });
   
   //        XposedHelpers.findAndHookMethod("com.example.targethook.MainActivity", lpparam.classLoader, "onCreate", Bundle.class, new XC_MethodHook() {
   //            @Override
   //            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
   //                XposedBridge.log("mylog_before");
   //            }
   //
   //            @Override
   //            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
   //                XposedBridge.log("mylog_after");
   //            }
   //        });
       }
   }
   ```

5. **创建程序入口**

   在 `app/src/main` 文件夹下新建一个 `assets` 文件夹，再在 `assets` 文件夹下创建 `xposed_init` 文件，在里面输入所编写的 `Xposed` 模块的完整包名，例如本项目即为 `com.example.alittle.hooktarget.Main` 

6. **Note** ：应将 `file >> setting >> Build, Execution, Deployment >> Instant Run` 中 

   - [x] `Enable Instant Run to hot swap code/resource changes on changes on deploy (default enable)` 前面的勾去掉