# Gson的使用

## 导入

在 Android Studio 中导入 Gson

> File > Project Structure > add Libary Dependencies > gson

```java
public class {

}
```



## 将对象转为JSON格式的字符串

```Java
Gson gson = new Gson();
Person person = new Person();
person.setName("奥特曼");
person.setAge(99999);
String jsonStr=gson.toJson(person);
String jsonStrWithType=gson.toJson(person);
```

Gson提供`toJson()`方法将对象转换成Json字符串。

## 将List转为JSON格式的字符串


