# <span style="color:yellow">高阶函数</span>

高阶函数 以另一个函数作为参数或者返回值的函数，在Kotlin中，函数可以用Lambda或者函数引用来表示,因此,任何Lambda或者函数引用作为参数或者返回值的函数,都可以视为**高阶函数**，例如
```kotlin
list.filter{ x>0 }
```

## 函数类型

```kotlin
val sum = {x:Int,y:Int -> x+y}
val action = {println(100)}
```
上面的例子中，sum和action都是函数类型，这里使用lambda表达式把函数类型存在变量中，变量并没有显式声明类型，而是由编译器来自动推断类型，显式声明如下：
```kotlin
val sum:(Int,Int)->Int = {x,y->x+1}
val action:()->Unit = {println(100)}
```
*Unit* 表示不返回任何值，通常省略，但是在这里需要显式声明，不可省略
**声明函数类型，需要将函数参数类型放在括号中，然后是一个箭头和函数返回类型**
```kotlin
(type,type,...) -> type
```
函数类型可以是**可空变量**
```kotlin
val sum:((Int,Int)->Int)? = null
```
注意和**可空返回值**写法上的不同
```kotlin
val sum:(Int,Int)->Int? = null
```
函数类型中的参数可以声明变量名，形如：
```kotlin
fun request(callback:(code:Int,content:String)->Unit){...}
request{code,content->()}
```
### 调用作为参数的函数
对于标准库扩展函数*String.filter*
```kotlin
fun String.filter(predicate:(Char)->Boolean):String{
    val sb = StringBuilder()
    for(index in 0 utils length){
        if(predicate(element)) sb.append(element)
    }
    return sb.toString()
}
```
调用该函数
```kotlin
"abc".filter{ it in 'a'..'z'}
```
上述例子中，*it*表示函数参数的输入参数*Char*，而 *it in 'a'..'z'* 是函数体具体实现

**对于被调用方，只需要关注函数参数的类型和返回值类型即可，对于调用者，需要提供函数参数的实现**


### 在Java中使用Kotlin函数类
Kotlin标准库定义了一系列接口，这些接口对用不同参数数量的函数，如
```kotlin
public interface Function0<out R> : Function<R> {
    public operator fun invoke(): R
}
```
这里使用`operator`重载了`()`运算符，使`invoke`方法可以直接调用，形如：
```kotlin
fun main() {
    val func: Function0<String> = object : Function0<String> {
        override fun invoke(): String {
            return "Hello, Kotlin!"
        }
    }
    
    // 使用 invoke() 方法（显式调用）
    println(func.invoke()) // 输出：Hello, Kotlin!
    
    // 由于 invoke() 被标记为 operator，可以直接调用
    println(func()) // 输出：Hello, Kotlin!
}

```

Java中以接口的方式直接调用，如：
```java
Object obj = new Function0<Integer>() {
    @Override
        public Integer invoke() {
            return null;         
        }
};
```
需要注意，对于返回`Unit`的函数，Java需要显式返回，Kotlin中`Unit`类型是一个确切值，与`void`不同

### 函数类型默认值和null值
和一般函数参数相同，函数类型可以设置默认参数和null值

### 函数类型作为函数返回值


### 通过lambda去除重复代码

## 