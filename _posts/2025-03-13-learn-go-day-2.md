---
layout: post
title: "Go语言高效学习-语法与基础入门 (Day 2)"
date: "2025-03-13"
categories: "技术"
tags: "Go语言高效学习" 
thumb: "/assets/images/post/Learn-Go-full.png"
---

###### 针对NodeJS工程师的Go语言学习计划
###### 📅 阶段一：语法与基础入门（Days 1-3） 
###### 目标：掌握与NodeJS差异显著的语法和编程范式。 
###### Day 2：结构体与方法


![](/assets/images/post/Learn-Go-full.png)

## 🚀 Go语言高效学习计划（NodeJS工程师版）
目标：2周快速掌握核心概念，上手大型项目；后续深入高级特性

本文涉及的代码链接：[Github](https://github.com/SincereMa/Go-Learn)

## 知识点详解与对比

### 1. 结构体（struct）

*   **Go 的特点:**
    *   Go 的 `struct` 是一种值类型。这意味着当结构体变量被赋值或作为参数传递时，会发生数据的复制。这与 Node.js 中的对象（引用类型）不同。
    *   结构体字段必须显式定义其类型。这与 TypeScript 中的类型声明类似，但 Go 是静态类型语言，类型检查更严格。
    *   Go 没有类（class）的概念，结构体是组织数据的核心方式。
    *   可以使用匿名字段，但不常用。
    *  支持tag,常用与json序列化，数据验证

*   **与 Node.js/TypeScript 对象对比:**
    *   Node.js/TypeScript 对象是引用类型，赋值或传递时只复制引用，而非数据本身。
    *   TypeScript 虽然也支持类型定义，但其类型系统是可选的，而 Go 的类型系统是强制的。
    *   结构体可以嵌套

*   **示例:**

    ```go
    package main

    import "fmt"

    // 定义一个名为 User 的结构体
    type User struct {
        ID       int    `json:"id"`
        Name     string `json:"name"`
        Email    string `json:"email"`
    }
      // 定义一个名为 UserEmail 的结构体 包含User
    type UserEmail struct {
        User
        Email    string `json:"email"`
    }

    func main() {
        // 创建 User 结构体实例
        user1 := User{ID: 1, Name: "Alice", Email: "alice@example.com"}

        // 复制 user1 到 user2
        user2 := user1

        // 修改 user2 的 Name 字段
        user2.Name = "Bob"

        // 打印 user1 和 user2，观察值类型的特性
        fmt.Println("user1:", user1) // 输出: user1: {1 Alice alice@example.com}
        fmt.Println("user2:", user2) // 输出: user2: {1 Bob alice@example.com}
    
      	// 创建 UserEmail 结构体实例
	      user3 := UserEmail{User:user1, Email: "alice_email@example.com"}
        // 打印 user3
      	fmt.Println("user3:", user3) //user3: { {1 Alice alice@example.com} alice_email@example.com }
    }
    ```

### 2. 方法接收者（值接收者 vs 指针接收者）

*   **Go 的特点:**
    *   方法可以与结构体关联。
    *   方法接收者可以是值类型或指针类型。
    *   值接收者操作的是结构体的副本，而指针接收者操作的是结构体本身。
    *   何时使用指针接收者：
        *   需要修改结构体的字段值时。
        *   避免大型结构体的复制开销。
        *   保持方法接收者类型的一致性（某些方法可能需要修改结构体，那么所有方法都应该使用指针接收者）。

*   **与 Node.js/TypeScript 方法对比:**
    *   Node.js/TypeScript 中，方法总是通过 `this` 关键字隐式地引用对象本身（类似于 Go 的指针接收者）。
    *   Go 提供了更细粒度的控制，可以选择值接收者或指针接收者。

*   **示例:**

    ```go
    package main

    import "fmt"

    type User struct {
        ID    int
        Name  string
        Email string
    }

    func (u User) Display() {
        fmt.Printf("ID: %d, Name: %s, Email: %s\n", u.ID, u.Name, u.Email)
    }

    // 值接收者方法：不会修改原始结构体
    func (u User) MockName() {
        u.Name = "Mock"
    }

    // 指针接收者方法：可以修改原始结构体
    func (u *User) ChangeEmail(newEmail string) {
        u.Email = newEmail
    }

    func main() {
        user := User{ID: 1, Name: "Alice", Email: "alice@example.com"}

        // 调用值接收者方法
        user.MockName()
        // 观察 Name 是否改变
        user.Display() // 输出: ID: 1, Name: Alice, Email: alice@example.com

        // 调用指针接收者方法
        user.ChangeEmail("new_alice@example.com")
        // 观察 Email 是否改变
        user.Display() // 输出: ID: 1, Name: Alice, Email: new_alice@example.com
    }

    ```

### 3. 接口（interface）的隐式实现

*   **Go 的特点:**
    *   Go 的接口定义了一组方法的集合。
    *   任何类型只要实现了接口中定义的*所有*方法，就被视为实现了该接口，无需显式声明。这就是所谓的“鸭子类型”（Duck Typing）。
    *   接口是一种抽象类型，不能实例化。
    *   接口可以嵌套

*   **与 Node.js/TypeScript 接口对比:**
    *   TypeScript 的接口也定义了方法集合，但类型必须显式声明实现接口（使用 `implements` 关键字）。
    *   Go 的隐式实现更加灵活，减少了代码的耦合。

*   **示例:**

    ```go
    package main

    import "fmt"

    // 定义一个名为 Stringer 的接口
    type Stringer interface {
      String() string
    }

    // 定义一个名为 Stringer2 的接口 嵌套Stringer
    type Stringer2 interface {
      Stringer
      String2() string
    }

    type User struct {
      ID    int
      Name  string
      Email string
    }

    // User 类型实现了 Stringer 接口
    func (u User) String() string {
      return fmt.Sprintf("User ID: %d, Name: %s", u.ID, u.Name)
    }

    // PrintStringer 函数接受任何实现了 Stringer 接口的类型
    func PrintStringer(s Stringer) {
      fmt.Println(s.String())
    }

    // PrintStringer2 函数接受任何实现了 Stringer2 接口的类型
    func PrintStringer2(s Stringer2) {
      fmt.Println(s.String2())
    }

    func main() {
      user := User{ID: 1, Name: "Alice", Email: "alice@example.com"}

      // User 类型隐式地实现了 Stringer 接口，可以传递给 PrintStringer 函数
      PrintStringer(user) // 输出: User ID: 1, Name: Alice

      // User 类型没有实现 Stringer2 接口，因此不可以传递给 PrintStringer2 函数
      // PrintStringer2(user)
    }
    ```

## 实战：用户模块（结构体 + CRUD 方法）

现在，我们将通过一个实战案例来综合运用以上知识点。我们将创建一个用户模块，包含用户结构体以及创建、读取、更新和删除用户的方法。

### Go 实现

```go
package main

import (
	"errors"
	"fmt"
)

// User 结构体定义
type User struct {
	ID    int
	Name  string
	Email string
}

// 定义错误信息
var (
	ErrUserNotFound = errors.New("user not found")
	ErrInvalidUser  = errors.New("invalid user data")
)

// users  slice 切片存储所有用户（模拟数据库）
var users []User

// nextID 用于生成唯一的自增用户 ID
var nextID = 1

// CreateUser 创建新用户（指针接收者，修改 users 列表）
func (u *User) CreateUser() error {
	// 简单验证
	if u.Name == "" || u.Email == "" {
		return ErrInvalidUser
	}

	u.ID = nextID
	nextID++
	users = append(users, *u) // 注意：这里需要存储 u 的副本，而不是指针
	return nil
}

// GetUserByID 根据 ID 获取用户（值接收者，不修改数据）
func GetUserByID(id int) (User, error) {
	for _, u := range users {
		if u.ID == id {
			return u, nil
		}
	}
	return User{}, ErrUserNotFound
}

// UpdateUser 更新用户信息（指针接收者，修改 users 列表中的用户）
func (u *User) UpdateUser() error {
	if u.Name == "" || u.Email == "" {
		return ErrInvalidUser
	}

	for i, existingUser := range users {
		if existingUser.ID == u.ID {
			users[i] = *u // 更新用户信息
			return nil
		}
	}
	return ErrUserNotFound
}

// DeleteUserByID 根据 ID 删除用户（修改 users 列表）
func DeleteUserByID(id int) error {
	for i, u := range users {
		if u.ID == id {
			// 从 users 切片中移除元素
			users = append(users[:i], users[i+1:]...)
			return nil
		}
	}
	return ErrUserNotFound
}

func main() {
	// 创建用户
	user1 := User{Name: "Alice", Email: "alice@example.com"}
	err := user1.CreateUser()
	if err != nil {
		fmt.Println("Error creating user:", err)
	}

	user2 := User{Name: "Bob", Email: "bob@example.com"}
	err = user2.CreateUser()
	if err != nil {
		fmt.Println("Error creating user:", err)
	}

	// 获取用户
	retrievedUser, err := GetUserByID(user1.ID)
	if err != nil {
		fmt.Println("Error getting user:", err)
	} else {
		fmt.Println("Retrieved user:", retrievedUser)
	}

	// 更新用户
	user1.Name = "Alice Updated"
	err = user1.UpdateUser()
	if err != nil {
		fmt.Println("Error updating user:", err)
	}

	// 再次获取用户，验证更新
	retrievedUser, err = GetUserByID(user1.ID)
	if err != nil {
		fmt.Println("Error getting user:", err)
	} else {
		fmt.Println("Retrieved user after update:", retrievedUser)
	}

	// 删除用户
	err = DeleteUserByID(user2.ID)
	if err != nil {
		fmt.Println("Error deleting user:", err)
	}

	// 尝试获取已删除的用户
	_, err = GetUserByID(user2.ID)
	if err == ErrUserNotFound {
		fmt.Println("User successfully deleted")
	}
}
```

希望以上详细的解释和代码示例能帮助您更好地理解 Go 语言的结构体、方法和接口，并将其与您熟悉的 Node.js/TypeScript 进行对比。如果您有任何其他问题，请随时提出。
