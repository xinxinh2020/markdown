[TOC]

## 基础知识

### 模型

gorm操作的数据表为结构体的小写名（或其复数形式）

结构体中的字段打上'-'标签表示忽略该字段

gorm.Model结构体，包括字段ID，CreatedAt，UpdatedAt，DeletedAt，可以继承它来获得这些字段：

- 字段ID默认为主键

- 字段UpdatedAt用于存储记录的修改时间，保存具有UpdatedAt字段的记录将被设置为当前时间

- 字段CreatedAt用于存储记录的创建时间，创建具有CreatedAt字段的记录将被设置为当前时间

- 字段DeletedAt用于存储记录的删除时间，删除具有DeletedAt字段的记录，它不会冲数据库中删除，但只将字段DeletedAt设置为当前时间，并在查询时无法找到记录

可以通过定义结构体的TableName()方法来显式指定表名

当结构体中包含另外一个结构体的变量时，两者可以有相互关联的关系，可以在结构体变量后面的标签中指定代表外键的字段



## 使用示例

```go
// 连接mysql数据库
db, err := gorm.Open("mysql", "root:password@tcp(localhost:3306)/mydb?charset=utf8&parseTime=True&loc=Local")

// 禁用表名复数，默认表名是结构体名称的复数形式
db.SingularTable(true)

// 检查表是否存在
db.HasTable("users")
db.HasTable(&User{}) // 推断表名

// 创建表，可以通过该gorm标签设置字段属性和约束
type User struct {
    UserId int  `gorm:"primary_key"`
    Phone string
    WxopenId string
    Tcreate *time.Time
    Tprocess *time.Time
    Balance int
    Src string
    Level int
}
db.CreateTable(&User{})

// 删除表
db.DropTable(&User{})
db.DropTable("user")
db.DropTableIfExists("user")

// 删除列
db.Model(&User{}).DropColumn("src")

<<<<<<< HEAD
// 关联查询
db.Model(&user).Related(&profile, "Profile") // SELECT * FROM profiles WHERE id = 111; // 111是user的外键ProfileID,如果字段名字和变量类型名(Profile)相同的话，后面的Profile可以省略
db.Model(&user).Related(&emails) // SELECT * FROM emails WHERE user_id = 111; 查询user对应的Email切片的值


db.NewRecord(&user) // 判断User的主键是否为空（是的话可以安全地插进数据库，不是的话也可以插入数据库，但可能会出现主键冲突的错误）
db.Create(&user) // 插入一条记录

// 查询
db.First(&user) // SELECT * FROM users ORDER BY id LIMIT 1; 获取第一条记录，按主键排序
db.Last(&user)  // SELECT * FROM users ORDER BY id DESC LIMIT 1; 获取最后一条记录
db.Find(&users) // SELECT * FROM users; 获取所有记录
db.First(&user, 10) // SELECT * FROM users WHERE id = 10; 使用主键获取记录
db.Where("name = ?", "jinzhu").First(&user) // SELECT * FROM users WHERE name = 'jinzhu' limit 1;
db.Where("name = ?", "jinzhu").Find(&users) // SELECT * FROM users WHERE name = 'jinzhu'; 获取所有匹配记录。注意这里的users要传切片的地址，切片需要先分配内存，大小随意，只要分了就行
db.Where("name <> ?", "jinzhu").Find(&users) // 不等于？？？


// IN
db.Where("name in (?)", []string{"jinzhu", "jinzhu 2"}).Find(&users)
// LIKE
db.Where("name LIKE ?", "%jin%").Find(&users)
// AND
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
// Time
db.Where("updated_at > ?", lastWeek).Find(&users)
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)

// Struct，只查询有值的字段
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
//// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 LIMIT 1;

// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
//// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;

// 主键的Slice
db.Where([]int64{20, 21, 22}).Find(&users)
//// SELECT * FROM users WHERE id IN (20, 21, 22);

//not
db.Not("name", "jinzhu").First(&user)
//// SELECT * FROM users WHERE name <> "jinzhu" LIMIT 1;

// Not In
db.Not("name", []string{"jinzhu", "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");

// Not In slice of primary keys
db.Not([]int64{1,2,3}).First(&user)
//// SELECT * FROM users WHERE id NOT IN (1,2,3);

db.Not("name = ?", "jinzhu").First(&user)
//// SELECT * FROM users WHERE NOT(name = "jinzhu");

// Struct
db.Not(User{Name: "jinzhu"}).First(&user)
//// SELECT * FROM users WHERE name <> "jinzhu";

// 增加记录
db.NewRecord(&user) # 判断User的主键是否为空（是的话可以安全地插进数据库，不是的话也可以插入数据库，但可能会出现主键冲突的错误）
db.Create(&user) # 插入一条记录

### 修改记录
db.Save(&user) # 设置所有字段，即使该字段的值没有发生变化
db.Model(&user).Update("name", "hello") # 更新单个字段，仅在该字段发生变化时才会执行更新操作
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "actived": false})  # 更新多个字段，仅会更新发生变化的字段

### 查询记录
# 关联查询
db.Model(&user).Related(&profile, "Profile") # SELECT * FROM profiles WHERE id = 111; // 111是user的外键ProfileID,如果字段名字和变量类型名(Profile)相同的话，后面的Profile可以省略
db.Model(&user).Related(&emails) # SELECT * FROM emails WHERE user_id = 111; 查询user对应的Email切片的值

db.First(&user) # SELECT * FROM users ORDER BY id LIMIT 1; 获取第一条记录
db.Last(&user)  # SELECT * FROM users ORDER BY id DESC LIMIT 1; 获取最后一条记录
db.Find(&users) # SELECT * FROM users; 获取所有记录
db.First(&user, 10) # SELECT * FROM users WHERE id = 10; 使用主键获取记录
db.Where("name = ?", "jinzhu").First(&user) # SELECT * FROM users WHERE name = 'jinzhu' limit 1;
db.Where("name = ?", "jinzhu").Find(&users) # SELECT * FROM users WHERE name = 'jinzhu'; 注意这里的users要传切片的地址，切片需要先分配内存，大小随意，只要分了就行

// 删除一条记录
database.DB.Delete(&entity.WorkFlowInstance{}, "id = ?", workflowInstanceId)
```



## 扩展方法

```shell
func (u User) TableName() string # 为User设置对应的表名
func (user *User) BeforeCreate(scope *gorm.Scope) error # 调Create方法之前的回调，可以在这里显式设置主键的值
```



## 标签

```shell
default // 似乎可以设置字段的默认值（未测试成功）
func (user *User) BeforeCreate(scope *gorm.Scope) error // 调Create方法之前的回调，可以在这里显式设置主键
```

