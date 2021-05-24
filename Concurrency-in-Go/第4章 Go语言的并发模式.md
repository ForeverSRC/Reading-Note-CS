# 第4章 Go语言的并发模式

## for-select循环

两种常见情况：

**1.向channel发送迭代变量**

```go
for _,s:=range []string{"a","b","c"} {
  select{
    case <-done:
    	return
    case stringStream <-s:
  }
}
```

**2.循环等待停止**

```go
for{
  select{
  case <-done:
    return
  default:
    //....
  }
}
```

## 防止goroutine泄漏

goroutine需要消耗资源，且goroutine不会被runtime GC。

goroutine有以下几种方式被终止：

* 完成了其工作
* 因不可恢复的错误而终止工作
* 被告知需要终止工作

