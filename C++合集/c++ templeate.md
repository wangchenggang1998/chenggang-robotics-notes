C++模板(Template)是C++中实现泛型编程的核心机制，允许编写与类型无关的代码，提高代码的复用性和类型安全性。

## 1. 函数模板 (Function Template)

### 基本语法

```cpp
template<typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}

// 使用
int main() {
    std::cout << max(3, 5) << std::endl;        // int版本
    std::cout << max(3.14, 2.71) << std::endl;  // double版本
    std::cout << max('a', 'z') << std::endl;    // char版本
    return 0;
}
```

### 多个模板参数

```cpp
template<typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}

// C++14及以后可以简化为
template<typename T, typename U>
auto add(T a, U b) {
    return a + b;
}

// 使用
auto result1 = add(3, 4.5);      // int + double = double
auto result2 = add(2.5f, 10);    // float + int = float
```

### 模板特化

```cpp
// 通用模板
template<typename T>
void print(T value) {
    std::cout << "Generic: " << value << std::endl;
}

// 特化版本
template<>
void print<const char*>(const char* value) {
    std::cout << "String: " << value << std::endl;
}

template<>
void print<bool>(bool value) {
    std::cout << "Boolean: " << (value ? "true" : "false") << std::endl;
}
```

## 2. 类模板 (Class Template)

### 基本类模板

```cpp
template<typename T>
class Stack {
private:
    std::vector<T> elements;
    
public:
    void push(const T& element) {
        elements.push_back(element);
    }
    
    void pop() {
        if (!elements.empty()) {
            elements.pop_back();
        }
    }
    
    T top() const {
        if (!elements.empty()) {
            return elements.back();
        }
        throw std::runtime_error("Stack is empty");
    }
    
    bool empty() const {
        return elements.empty();
    }
    
    size_t size() const {
        return elements.size();
    }
};

// 使用
Stack<int> intStack;
Stack<std::string> stringStack;
```

### 多个模板参数的类模板

```cpp
template<typename Key, typename Value, typename Compare = std::less<Key>>
class Map {
private:
    std::map<Key, Value, Compare> data;
    
public:
    void insert(const Key& key, const Value& value) {
        data[key] = value;
    }
    
    Value& operator[](const Key& key) {
        return data[key];
    }
    
    bool contains(const Key& key) const {
        return data.find(key) != data.end();
    }
    
    void print() const {
        for (const auto& pair : data) {
            std::cout << pair.first << ": " << pair.second << std::endl;
        }
    }
};

// 使用
Map<std::string, int> nameAge;
Map<int, std::string, std::greater<int>> reversedMap;  // 使用自定义比较器
```

## 3. 非类型模板参数

```cpp
template<typename T, size_t N>
class Array {
private:
    T data[N];
    
public:
    T& operator[](size_t index) {
        return data[index];
    }
    
    const T& operator[](size_t index) const {
        return data[index];
    }
    
    constexpr size_t size() const {
        return N;
    }
    
    // 迭代器支持
    T* begin() { return data; }
    T* end() { return data + N; }
    const T* begin() const { return data; }
    const T* end() const { return data + N; }
};

// 使用
Array<int, 5> arr;
Array<double, 10> doubleArr;
```

## 4. 模板特化

### 全特化

```cpp
template<typename T>
class TypeInfo {
public:
    static std::string name() { return "Unknown"; }
    static size_t size() { return sizeof(T); }
};

// 全特化
template<>
class TypeInfo<int> {
public:
    static std::string name() { return "Integer"; }
    static size_t size() { return sizeof(int); }
};

template<>
class TypeInfo<double> {
public:
    static std::string name() { return "Double"; }
    static size_t size() { return sizeof(double); }
};
```

### 偏特化

```cpp
// 通用模板
template<typename T, typename U>
class Pair {
private:
    T first;
    U second;
    
public:
    Pair(const T& f, const U& s) : first(f), second(s) {}
    
    void print() const {
        std::cout << "Generic pair: " << first << ", " << second << std::endl;
    }
};

// 偏特化：两个参数类型相同
template<typename T>
class Pair<T, T> {
private:
    T first;
    T second;
    
public:
    Pair(const T& f, const T& s) : first(f), second(s) {}
    
    void print() const {
        std::cout << "Same type pair: " << first << ", " << second << std::endl;
    }
    
    bool equal() const {
        return first == second;
    }
};

// 偏特化：指针类型
template<typename T>
class Pair<T*, T*> {
private:
    T* first;
    T* second;
    
public:
    Pair(T* f, T* s) : first(f), second(s) {}
    
    void print() const {
        std::cout << "Pointer pair: " << *first << ", " << *second << std::endl;
    }
};
```

## 5. 变参模板 (Variadic Templates) - C++11

```cpp
// 递归展开
template<typename T>
void print(T&& t) {
    std::cout << t << std::endl;
}

template<typename T, typename... Args>
void print(T&& t, Args&&... args) {
    std::cout << t << " ";
    print(args...);
}

// C++17折叠表达式
template<typename... Args>
void print_cpp17(Args&&... args) {
    ((std::cout << args << " "), ...);
    std::cout << std::endl;
}

// 变参模板类
template<typename... Args>
class Tuple;

template<>
class Tuple<> {
    // 空tuple
};

template<typename Head, typename... Tail>
class Tuple<Head, Tail...> : private Tuple<Tail...> {
private:
    Head head;
    
public:
    Tuple(Head h, Tail... t) : Tuple<Tail...>(t...), head(h) {}
    
    Head& getHead() { return head; }
    const Head& getHead() const { return head; }
    
    Tuple<Tail...>& getTail() { return *this; }
    const Tuple<Tail...>& getTail() const { return *this; }
};
```

## 6. 模板元编程

### 编译时计算

```cpp
template<int N>
struct Factorial {
    static constexpr int value = N * Factorial<N - 1>::value;
};

template<>
struct Factorial<0> {
    static constexpr int value = 1;
};

// 使用
constexpr int fact5 = Factorial<5>::value;  // 编译时计算
```

### SFINAE (Substitution Failure Is Not An Error)

```cpp
#include <type_traits>

template<typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
process(T value) {
    return value * 2;
}

template<typename T>
typename std::enable_if<std::is_floating_point<T>::value, T>::type
process(T value) {
    return value * 1.5;
}

// C++17 if constexpr
template<typename T>
auto process_cpp17(T value) {
    if constexpr (std::is_integral_v<T>) {
        return value * 2;
    } else if constexpr (std::is_floating_point_v<T>) {
        return value * 1.5;
    } else {
        return value;
    }
}
```

## 7. 现代C++模板特性

### 概念 (Concepts) - C++20

```cpp
#include <concepts>

template<typename T>
concept Arithmetic = std::integral<T> || std::floating_point<T>;

template<Arithmetic T>
T multiply(T a, T b) {
    return a * b;
}

// 更复杂的概念
template<typename T>
concept Printable = requires(T t) {
    std::cout << t;
};

template<Printable T>
void print(const T& value) {
    std::cout << value << std::endl;
}
```

### 缩写函数模板 - C++20

```cpp
// 传统写法
template<typename T>
void func(T param) { /* ... */ }

// C++20缩写写法
void func(auto param) { /* ... */ }

// 约束的缩写写法
void func(Arithmetic auto param) { /* ... */ }
```

## 8. 实际应用示例

### 智能指针实现

```cpp
template<typename T>
class unique_ptr {
private:
    T* ptr;
    
public:
    explicit unique_ptr(T* p = nullptr) : ptr(p) {}
    
    ~unique_ptr() {
        delete ptr;
    }
    
    // 禁止拷贝
    unique_ptr(const unique_ptr&) = delete;
    unique_ptr& operator=(const unique_ptr&) = delete;
    
    // 移动语义
    unique_ptr(unique_ptr&& other) noexcept : ptr(other.ptr) {
        other.ptr = nullptr;
    }
    
    unique_ptr& operator=(unique_ptr&& other) noexcept {
        if (this != &other) {
            delete ptr;
            ptr = other.ptr;
            other.ptr = nullptr;
        }
        return *this;
    }
    
    T& operator*() { return *ptr; }
    T* operator->() { return ptr; }
    T* get() const { return ptr; }
    
    T* release() {
        T* temp = ptr;
        ptr = nullptr;
        return temp;
    }
    
    void reset(T* p = nullptr) {
        delete ptr;
        ptr = p;
    }
};
```

### 线程安全的单例模板

```cpp
template<typename T>
class Singleton {
private:
    static std::once_flag flag;
    static std::unique_ptr<T> instance;
    
protected:
    Singleton() = default;
    
public:
    static T& getInstance() {
        std::call_once(flag, []() {
            instance = std::make_unique<T>();
        });
        return *instance;
    }
    
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
};

template<typename T>
std::once_flag Singleton<T>::flag;

template<typename T>
std::unique_ptr<T> Singleton<T>::instance;

// 使用
class MyClass : public Singleton<MyClass> {
    friend class Singleton<MyClass>;
private:
    MyClass() = default;
public:
    void doSomething() {
        std::cout << "Doing something..." << std::endl;
    }
};
```

## 最佳实践

1. **使用有意义的模板参数名**：`template<typename Iterator>` 而不是 `template<typename T>`
2. **提供默认模板参数**：简化使用
3. **使用概念约束模板参数**：提高代码可读性和错误信息
4. **避免过度复杂的模板**：保持代码可维护性
5. **使用模板别名**：简化复杂类型名
6. **合理使用SFINAE**：但优先考虑概念(C++20)

```cpp
// 模板别名
template<typename T>
using Vec = std::vector<T>;

template<typename K, typename V>
using Map = std::unordered_map<K, V>;

// 使用
Vec<int> numbers = {1, 2, 3, 4, 5};
Map<std::string, int> ages = {{"Alice", 25}, {"Bob", 30}};
```

C++模板是一个强大的工具，正确使用能够编写出高效、类型安全且可复用的代码。