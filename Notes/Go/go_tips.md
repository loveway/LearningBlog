# Go Tips
##### 1、编写测试程序注意
* 源码文件以 *_test* 结尾，如 `xxx_test.go` 这样，表示这是个测试文件
* 测试方法名以 *Test* 开头，如 `func TestXXX(t *testing.T) { ... }`
* 如果想保存时运行 test ，在 setting.json 中设置 ` "go.coverOnSave": true`, 如果vscode没有输出 可以再加上 `"go.testFlags": ["-v", "-count=1"]`

##### 2、赋值

斐波那契数列

```go
import (
	"fmt"
	"testing"
)
func TestFibList(t *testing.T) {
	a := 1
	b := 3
	for i := 0; i < 6; i++ {
		fmt.Println(b)
		tmp := a
		a = b
		b = a + tmp
	}
}
```
可简化为
```go
import (
	"fmt"
	"testing"
)
func TestFibList(t *testing.T) {
	a := 1
	b := 3
	for i := 0; i < 6; i++ {
		fmt.Println(b)
		a, b = b, a+b
	}
}
```
##### 3、Go 不支持隐式类型转换
```go
import (
	"testing"
)
func TestImp(t *testing.T) {
	a := 0
	var b int64 = 1
	b = int64(a)//如果直接写 b = a 就报错
	t.Log(a, b)
}
```
##### 4、类型别名
`type myString string`
##### 5、指针
指针不能直接进行计算, 字符串申明时没赋值，初始值是 "" ，如下 `s = ""`
```go
func TestPoint(t *testing.T) {
	var s string
	t.Log("s is", s)
	s = "loveway"
	pS := &s + 1// error "invalid operation: &s + 1 (mismatched types *string and int)"
}
```

