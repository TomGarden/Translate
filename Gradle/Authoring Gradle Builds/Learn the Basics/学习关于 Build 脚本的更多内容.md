……







## Extra properties (额外属性)

Gradle 模式中的所有增强对象都能包含额外的用户定义属性 。
这被包含，但是不强制 project，task 和 source 使用它 。

Extar 属性可以被添加读取通过项目的 `ext` 属性。
另外一个 `ext` 语句块可以被一次性添加多个属性。

```Groovy
//build.gradle
plugins {
    id 'java'
}

ext {
    springVersion = "3.1.0.RELEASE"
    emailNotification = "build@master.org"
}

sourceSets.all { ext.purpose = null }

sourceSets {
    main {
        purpose = "production"
    }
    test {
        purpose = "test"
    }
    plugin {
        purpose = "production"
    }
}

task printProperties {
    doLast {
        println springVersion
        println emailNotification
        sourceSets.matching { it.purpose == "production" }.each { println it.name }
    }
}
```

```terminate
Output of gradle -q printProperties
> gradle -q printProperties
3.1.0.RELEASE
build@master.org
main
plugin
```

在这个例子中，一个 ext 语句块增加了两个额外属性到 `project` 对象中。
另外，一个属性名 `purpose` 通过设置 `ext.purpose` 为 `null`(`null` 是一个被允许的值) 被增加到每一个 source set 中。
一旦属性被增加，它就可以被读取和写入，就像那些预定义的属性 。

通过使用特定的语法增加属性， 当试图去设置一个(预定义或者额外的)属性但是该属性丢失或者不存在的时候， Gradle 会快速失败。
额外属性能在拥有其对象的任何位置访问它们，从而提供了比局部变量更宽泛的访问范围。
一个 project 的额外属性可以在所有的 子 project 中被访问 。

额外属性的功能详情查看 [ ExtraPropertiesExtension ](https://docs.gradle.org/current/dsl/org.gradle.api.plugins.ExtraPropertiesExtension.html)






……