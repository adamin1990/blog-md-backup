# Flutter 集成到现有的安卓原生项目中

> Flutter可以作为Gradle子工程或AAR文件嵌入到现有的安卓原生项目中，集成流程可以使用Android Studio的插件或手动集成

~~~
注意：你的原生安卓项目可能支持mips或x86架构，Flutter目前（2021年9月）只支持AOT模式编译x86_64,armabi-v7a,arm64-v8a
可以在app上使用abiFilters安卓Gradle插件api限制支持的架构，避免缺少libflutter.so造成的运行时异常，比如：
android {
  //...
  defaultConfig {
    ndk {
      // Filter for architectures supported by Flutter.
      abiFilters 'armeabi-v7a', 'arm64-v8a', 'x86_64'
    }
  }
}
Flutter引用有x86和x86——64版本，当使用模拟器JIT模式调试时，Flutter模块会正确的运行。
~~~

