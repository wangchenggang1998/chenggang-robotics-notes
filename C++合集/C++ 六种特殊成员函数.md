C++中的六种特殊成员函数是编译器在特定条件下自动生成的成员函数。这些函数在C++11标准中得到了完善：

## 1. 默认构造函数 (Default Constructor)

**功能**：创建对象时不需要参数的构造函数

cpp

```cpp
class MyClass {
public:
    MyClass() = default;  // 显式要求编译器生成
    // 或者自定义
    MyClass() : data(0) {}
    
private:
    int data;
};
```

**自动生成条件**：

- 没有定义任何构造函数时
- 所有成员都有默认构造函数或内置类型

## 2. 析构函数 (Destructor)

**功能**：对象销毁时自动调用，释放资源

cpp

```cpp
class MyClass {
public:
    ~MyClass() = default;  // 编译器生成
    // 或者自定义
    ~MyClass() {
        delete[] data;
    }
    
private:
    int* data;
};
```

**自动生成条件**：

- 总是会生成（除非显式删除）
- 默认为public且非虚函数

## 3. 拷贝构造函数 (Copy Constructor)

**功能**：用同类型对象初始化新对象

cpp

```cpp
class MyClass {
public:
    // 编译器生成的版本
    MyClass(const MyClass& other) = default;
    
    // 自定义版本
    MyClass(const MyClass& other) : data(new int(*other.data)) {}
    
private:
    int* data;
};
```

**自动生成条件**：

- 没有显式定义时
- 没有定义移动构造函数和移动赋值运算符

## 4. 拷贝赋值运算符 (Copy Assignment Operator)

**功能**：将一个对象的值赋给另一个已存在的对象

cpp

```cpp
class MyClass {
public:
    // 编译器生成的版本
    MyClass& operator=(const MyClass& other) = default;
    
    // 自定义版本
    MyClass& operator=(const MyClass& other) {
        if (this != &other) {
            delete data;
            data = new int(*other.data);
        }
        return *this;
    }
    
private:
    int* data;
};
```

## 5. 移动构造函数 (Move Constructor) - C++11

**功能**：从右值对象"移动"资源来构造新对象

cpp

```cpp
class MyClass {
public:
    // 编译器生成的版本
    MyClass(MyClass&& other) = default;
    
    // 自定义版本
    MyClass(MyClass&& other) noexcept 
        : data(other.data) {
        other.data = nullptr;  // 原对象置空
    }
    
private:
    int* data;
};
```

**自动生成条件**：

- 没有显式定义拷贝构造函数、拷贝赋值运算符或析构函数
- 所有成员都支持移动操作

## 6. 移动赋值运算符 (Move Assignment Operator) - C++11

**功能**：从右值对象"移动"资源到当前对象

cpp

```cpp
class MyClass {
public:
    // 编译器生成的版本
    MyClass& operator=(MyClass&& other) = default;
    
    // 自定义版本
    MyClass& operator=(MyClass&& other) noexcept {
        if (this != &other) {
            delete data;
            data = other.data;
            other.data = nullptr;
        }
        return *this;
    }
    
private:
    int* data;
};
```

## 三/五/零法则

**三法则 (Rule of Three)**： 如果需要自定义析构函数、拷贝构造函数或拷贝赋值运算符中的任何一个，通常需要定义所有三个。

**五法则 (Rule of Five)**： C++11后，如果需要自定义上述三个中的任何一个，通常也需要考虑移动构造函数和移动赋值运算符。

**零法则 (Rule of Zero)**： 尽量避免自定义这些特殊成员函数，使用RAII和智能指针等现代C++技术。

## 完整示例

cpp

```cpp
class Resource {
private:
    int* data;
    size_t size;
    
public:
    // 1. 默认构造函数
    Resource() : data(nullptr), size(0) {}
    
    // 2. 析构函数
    ~Resource() { delete[] data; }
    
    // 3. 拷贝构造函数
    Resource(const Resource& other) 
        : size(other.size), data(new int[size]) {
        std::copy(other.data, other.data + size, data);
    }
    
    // 4. 拷贝赋值运算符
    Resource& operator=(const Resource& other) {
        if (this != &other) {
            delete[] data;
            size = other.size;
            data = new int[size];
            std::copy(other.data, other.data + size, data);
        }
        return *this;
    }
    
    // 5. 移动构造函数
    Resource(Resource&& other) noexcept 
        : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;
    }
    
    // 6. 移动赋值运算符
    Resource& operator=(Resource&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
};
```

理解这六种特殊成员函数对于编写高效、安全的C++代码至关重要。