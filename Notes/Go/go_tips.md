# Go Tips
1、编写测试程序注意
* 源码文件以 _test 结尾，如xxx_test.go 这样，表示这是个测试文件
* 测试方法名以 Test 开头，如 func TestXXX(t *testing.T) { ... }
* 如果想保存时运行 test ，在 setting.json 中设置 ` "go.coverOnSave": true`, 如果vscode没有输出 可以再加上 `"go.testFlags": ["-v", "-count=1"]`

2、赋值

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
3、Go 不支持隐式类型转换
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