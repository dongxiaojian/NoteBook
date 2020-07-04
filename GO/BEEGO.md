1. 安装beego：` go get github.com/astaxie/beego`
2. 安装beego的工具： `go get github.com/beego/bee`
3. 安装orm的：`go get github.com/astaxie/beego/orm`
4. 安装mysql驱动：` go get -u  github.com/go-sql-driver/mysql`

##### 2. model相关

```go
// 初始化表
func init() {
	orm.RegisterDataBase("default", "mysql", "root:123456@tcp(127.0.0.1:3306)/gotest?charset=utf8")
	orm.RegisterModel(new(User))
	orm.RunSyncdb("default", false, true)
}

```

1. 创建model字段的时候，开头的的大写会自动转换成小写； 中间的大写，会转换成下划线+小写字母
2. 关于数据库时间问题， 需要在后边添加上`&loc=Local`才能显示本地时间。
3. golang数字转换成字符串用`strconv.Itoa`
4. 



==问题点：1. 如何通过beego的orm修改列名而不用重新创建表？？==

#### 3. controllers相关


test-dzt 001
test-dzt 002

