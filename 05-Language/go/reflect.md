### 反射 reflect

##### 概念：

         程序编译时，变量转换成内存地址，变量名不会编译到程序的可执行部分，运行程序时，程序无法获取自身的信息。

        支持反射的语言在程序编译的时候，将变量的反射信息，如字段名，类型信息，结构体信息等整合到可执行文件中，并给程序提供访问反射信息的接口，这样就可以在程序运行的时候获取反射信息，并修改他们

        go语言运行时候通过reflect包获取反射信息。

##### 获取类型信息

```go
package main

import (
    "fmt"
    "reflect"
)

type Student struct {

    Name string
    Age  int
}

func main() {

    var stu Student

    typeOfStu := reflect.TypeOf(stu)

    fmt.Println(typeOfStu.Name(), typeOfStu.Kind())
}
代码输出：
Student struct
```

##### 通过反射获取指针的元素类型

```go
package main

import (
    "fmt"
    "reflect"
)

type Student struct {

    Name string
    Age  int
}

func main() {

    //定义一个Student类型的指针变量
    var stu = &Student{Name:"kitty", Age: 20}

    //获取结构体实例的反射类型对象
    typeOfStu := reflect.TypeOf(stu)

    //显示反射类型对象的名称和种类
    fmt.Printf("name: '%v', kind: '%v'\n", typeOfStu.Name(), typeOfStu.Kind())

    //取类型的元素
    typeOfStu = typeOfStu.Elem()

    //显示反射类型对象的名称和种类
    fmt.Printf("element name: '%v', element kind: '%v'\n", typeOfStu.Name(), typeOfStu.Kind())
}
输出：
name: '', kind: 'ptr'
element name: 'Student', element kind: 'struct'
```

##### 通过反射获取结构体成员的类型

| -方法                                                          | -说明                                                                  |
| ------------------------------------------------------------ | -------------------------------------------------------------------- |
| Field(i int) StructField                                     | 根据索引，返回索引对应的结构体字段的信息。当值不是结构体或索引超界时发生宕机                               |
| NumField() int                                               | 返回结构体成员字段数量。当类型不是结构体或索引超界时发生宕机                                       |
| FieldByName(name string) (StructField, bool)                 | 根据给定字符串返回字符串对应的结构体字段的信息。没有找到时 bool 返回 false，当类型不是结构体或索引超界时发生宕机       |
| FieldByIndex(index []int) StructField                        | 多层成员访问时，根据 []int 提供的每个结构体的字段索引，返回字段的信息。没有找到时返回零值。当类型不是结构体或索引超界时 发生宕机 |
| FieldByNameFunc( match func(string) bool) (StructField,bool) | 根据匹配函数匹配需要的字段。当值不是结构体或索引超界时发生宕机                                      |

```go
type StructField struct {
    Name string          // 字段名
    PkgPath string       // 字段路径
    Type      Type       // 字段反射类型对象
    Tag       StructTag  // 字段的结构体标签
    Offset    uintptr    // 字段在结构体中的相对偏移
    Index     []int      // Type.FieldByIndex中的返回的索引值
    Anonymous bool       // 是否为匿名字段
}
package main

import (
    "fmt"
    "reflect"
)

func main() {

    // 声明一个空结构体
    type cat struct {
        Name string

        // 带有结构体tag的字段
        Type int `json:"type" id:"100"`
    }

    // 创建cat的实例
    ins := cat{Name: "mimi", Type: 1}

    // 获取结构体实例的反射类型对象
    typeOfCat := reflect.TypeOf(ins)

    // 遍历结构体所有成员
    for i := 0; i < typeOfCat.NumField(); i++ {

        // 获取每个成员的结构体字段类型
        fieldType := typeOfCat.Field(i)

        // 输出成员名和tag
        fmt.Printf("name: %v  tag: '%v'\n", fieldType.Name, fieldType.Tag)
    }

    // 通过字段名, 找到字段类型信息
    if catType, ok := typeOfCat.FieldByName("Type"); ok {

        // 从tag中取出需要的tag
        fmt.Println(catType.Tag.Get("json"), catType.Tag.Get("id"))
    }
}
代码输出：
name: Name  tag: ''
name: Type  tag: 'json:"type" id:"100"'
type 100
```

##### 通过反射获取值的信息

```go
package main

import (
    "fmt"
    "reflect"
)

//定义结构体
type Student struct {
    Name string
    Age  int

    //嵌入字段
    float32
    bool


    next *Student
}

func main() {

    //值包装结构体
    rValue := reflect.ValueOf(Student{
        next: &Student{},
    })

    //获取字段数量
    fmt.Println("NumField:", rValue.NumField())

    //获取索引为2的字段(float32字段)
    //注:经过测试发现Field(i)的参数索引是从0开始的，
    //并且是按照定义的结构体的顺序来的，而不是按照字段名字的ASCii码值来的
    floatField := rValue.Field(2)

    //输出字段类型
    fmt.Println("Field:", floatField.Type())

    //根据名字查找字段
    fmt.Println("FieldByName(\"Age\").Type:", rValue.FieldByName("Age").Type())

    //根据索引查找值中next字段的int字段的值
    fmt.Println("FieldByIndex([]int{4, 0}).Type()", rValue.FieldByIndex([]int{4, 0}).Type())

}
输出信息：

NumField: 5
Field: float32
FieldByName("Age").Type: int
FieldByIndex([]int{4, 0}).Type() string
```

第 41 行，[]int{4,0} 中的 4 表示，在 Student 结构中索引值为 4 的成员，也就是 next。next 的类型为 Student，也是一个结构体，因此使用 []int{4,0} 中的 0 继续在 next 值的基础上索引，结构为 Student 中索引值为 0 的 Name 字段，类型为 string。

##### 判断反射值的空和有效性

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {

    //*int的空指针
    var a *int
    fmt.Println("var a *int:", reflect.ValueOf(a).IsNil())

    //nil值
    fmt.Println("nil:", reflect.ValueOf(nil).IsValid())

    //*int类型的空指针
    fmt.Println("(*int)(nil):", reflect.ValueOf((*int)(nil)).Elem().IsValid())

    //实例化一个结构体
    s := struct {}{}

    //尝试从结构体中查找一个不存在的字段
    fmt.Println("不存在的结构体成员:", reflect.ValueOf(s).FieldByName("").IsValid())

    //尝试从结构体中查找一个不存在的方法
    fmt.Println("不存在的方法:", reflect.ValueOf(s).MethodByName("").IsValid())

    //实例化一个map
    m := map[int]int{}

    //尝试从map中查找一个不存在的键
    fmt.Println("不存在的键:", reflect.ValueOf(m).MapIndex(reflect.ValueOf(3)).IsValid())
}

输出结果:
var a *int: true
nil: false
(*int)(nil): false
不存在的结构体成员: false
不存在的方法: false
不存在的键: false
```

##### 通过反射修改变量的值

```go
可被寻址：
package main

import (
    "fmt"
    "reflect"
)

func main() {

    //声明整形变量a并赋初值
    var a int = 1024

    //获取变量a的反射值对象
    rValue := reflect.ValueOf(&a)

    //取出a地址的元素(a的值)
    rValue = rValue.Elem()

    //尝试将a修改为1
    rValue.SetInt(1)

    //打印a的值
    fmt.Println(rValue.Int())
}

可被导出：
package main

import (
    "fmt"
    "reflect"
)

func main() {

    type dog struct {
        LegCount int
    }

    //获取dog实例的反射值对象
    valueOfDog := reflect.ValueOf(&dog{})

  //// 取出dog实例地址的元素
    valueOfDog = valueOfDog.Elem()

    //获取legCount字段的值
    vLegCount := valueOfDog.FieldByName("LegCount")

    //尝试设置legCount的值
    vLegCount.SetInt(4)

    fmt.Println(vLegCount.Int())
}
```

##### 通过类型信息创建实例

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {

    var a int

    //取变量a的反射类型对象
    typeOfA := reflect.TypeOf(a)

    //根据反射类型对象创建类型实例
    aIns := reflect.New(typeOfA)

    //输出Value的类型和种类
    fmt.Println(aIns.Type(), aIns.Kind())
}

代码输出结果如下

*int ptr
```

##### 通过反射调用函数

```go
package main

import (
    "fmt"
    "reflect"
)

//普通函数
func add(a, b int) int {
    return a + b
}

func main() {

    //将函数包装为反射值对象
    funcValue := reflect.ValueOf(add)

    //构造函数参数，传入两个整形值
    paramList := []reflect.Value{reflect.ValueOf(2), reflect.ValueOf(3)}

    //反射调用函数
    retList := funcValue.Call(paramList)

    fmt.Println(retList[0].Int())
}
```

##### 通过反射调用方法

```go
package main

import (
    "fmt"
    "reflect"
)

type MyMath struct {
    Pi float64
}

//普通函数
func (myMath MyMath) Sum(a, b int) int {
    return a + b
}

func (myMath MyMath) Dec(a, b int) int {
    return a - b
}

func main() {

    var myMath = MyMath{Pi:3.14159}

    //获取myMath的值对象
    rValue := reflect.ValueOf(myMath)

    //获取到该结构体有多少个方法
    //numOfMethod := rValue.NumMethod()

    //构造函数参数，传入两个整形值
    paramList := []reflect.Value{reflect.ValueOf(30), reflect.ValueOf(20)}


    //调用结构体的第一个方法Method(0)
    //注意:在反射值对象中方法索引的顺序并不是结构体方法定义的先后顺序
    //而是根据方法的ASCII码值来从小到大排序，所以Dec排在第一个，也就是Method(0)
    result := rValue.Method(0).Call(paramList)

    fmt.Println(result[0].Int())

}

代码输出结果为:
10
```
