C++元编程是在编译时执行的编程技术，允许程序操作程序本身，生成高效的运行时代码。它是C++模板系统的高级应用。

## 1. 基本概念

### 什么是元编程

元编程是"编写编写代码的代码"，在C++中主要通过模板系统实现，代码在编译时执行。

```cpp
// 编译时计算斐波那契数列
template<int N>
struct Fibonacci {
    static constexpr int value = Fibonacci<N-1>::value + Fibonacci<N-2>::value;
};

template<>
struct Fibonacci<0> {
    static constexpr int value = 0;
};

template<>
struct Fibonacci<1> {
    static constexpr int value = 1;
};

// 编译时计算，运行时直接使用结果
constexpr int fib10 = Fibonacci<10>::value;  // 55
```

## 2. 类型操作

### 类型特征 (Type Traits)

```cpp
// 自实现的类型特征
template<typename T>
struct is_pointer {
    static constexpr bool value = false;
};

template<typename T>
struct is_pointer<T*> {
    static constexpr bool value = true;
};

template<typename T>
struct is_const {
    static constexpr bool value = false;
};

template<typename T>
struct is_const<const T> {
    static constexpr bool value = true;
};

// 移除引用
template<typename T>
struct remove_reference {
    using type = T;
};

template<typename T>
struct remove_reference<T&> {
    using type = T;
};

template<typename T>
struct remove_reference<T&&> {
    using type = T;
};

// 使用
static_assert(is_pointer<int*>::value == true);
static_assert(is_const<const int>::value == true);
using type = remove_reference<int&>::type;  // int
```

### 条件类型选择

```cpp
template<bool Condition, typename TrueType, typename FalseType>
struct conditional {
    using type = TrueType;
};

template<typename TrueType, typename FalseType>
struct conditional<false, TrueType, FalseType> {
    using type = FalseType;
};

// 使用
template<typename T>
using storage_type = typename conditional
    sizeof(T) <= sizeof(void*),
    T,
    T*
>::type;

// 小对象直接存储，大对象存储指针
storage_type<int> small_storage;     // int
storage_type<std::string> big_storage; // std::string*
```

## 3. SFINAE (Substitution Failure Is Not An Error)

### 基本SFINAE

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

template<typename T>
typename std::enable_if<std::is_same<T, std::string>::value, T>::type
process(T value) {
    return value + value;
}
```

### 检测成员函数存在性

```cpp
template<typename T>
class has_serialize {
private:
    template<typename U>
    static auto test(int) -> decltype(std::declval<U>().serialize(), std::true_type{});
    
    template<typename>
    static std::false_type test(...);
    
public:
    static constexpr bool value = decltype(test<T>(0))::value;
};

// 使用
class A {
public:
    void serialize() { /* ... */ }
};

class B {
    // 没有serialize方法
};

static_assert(has_serialize<A>::value == true);
static_assert(has_serialize<B>::value == false);
```

## 4. 编译时计算

### 编译时数学运算

```cpp
template<int Base, int Exp>
struct Power {
    static constexpr int value = Base * Power<Base, Exp - 1>::value;
};

template<int Base>
struct Power<Base, 0> {
    static constexpr int value = 1;
};

template<int N>
struct Factorial {
    static constexpr int value = N * Factorial<N - 1>::value;
};

template<>
struct Factorial<0> {
    static constexpr int value = 1;
};

// 编译时素数检查
template<int N, int Divisor = N - 1>
struct IsPrime {
    static constexpr bool value = (N % Divisor != 0) && IsPrime<N, Divisor - 1>::value;
};

template<int N>
struct IsPrime<N, 1> {
    static constexpr bool value = true;
};

template<>
struct IsPrime<1, 1> {
    static constexpr bool value = false;
};

// 使用
constexpr int pow_2_8 = Power<2, 8>::value;  // 256
constexpr int fact_5 = Factorial<5>::value;  // 120
constexpr bool is_17_prime = IsPrime<17>::value;  // true
```

### 编译时字符串处理

```cpp
template<size_t N>
struct compile_time_string {
    char data[N];
    constexpr compile_time_string(const char (&str)[N]) {
        for (size_t i = 0; i < N; ++i) {
            data[i] = str[i];
        }
    }
};

template<compile_time_string str>
struct string_length {
    static constexpr size_t value = sizeof(str.data) - 1;
};

// C++20 使用
constexpr auto hello = compile_time_string("Hello");
constexpr size_t len = string_length<hello>::value;  // 5
```

## 5. 变参模板元编程

### 递归处理参数包

```cpp
// 计算参数包中的最大值
template<typename T>
constexpr T max_value(T&& val) {
    return val;
}

template<typename T, typename... Args>
constexpr T max_value(T&& val, Args&&... args) {
    auto rest_max = max_value(args...);
    return val > rest_max ? val : rest_max;
}

// 计算参数包大小
template<typename... Args>
struct pack_size {
    static constexpr size_t value = sizeof...(Args);
};

// 获取第N个类型
template<size_t N, typename... Args>
struct nth_type;

template<size_t N, typename Head, typename... Tail>
struct nth_type<N, Head, Tail...> {
    using type = typename nth_type<N - 1, Tail...>::type;
};

template<typename Head, typename... Tail>
struct nth_type<0, Head, Tail...> {
    using type = Head;
};

// 使用
using third_type = nth_type<2, int, double, char, float>::type;  // char
```

### 编译时列表操作

```cpp
template<typename... Types>
struct type_list {};

// 连接两个类型列表
template<typename List1, typename List2>
struct concat;

template<typename... Types1, typename... Types2>
struct concat<type_list<Types1...>, type_list<Types2...>> {
    using type = type_list<Types1..., Types2...>;
};

// 过滤类型
template<template<typename> class Predicate, typename List>
struct filter;

template<template<typename> class Predicate, typename... Types>
struct filter<Predicate, type_list<Types...>> {
    using type = decltype(std::tuple_cat(
        std::conditional_t<Predicate<Types>::value, 
                          std::tuple<Types>, 
                          std::tuple<>>{}...
    ));
};

// 转换为tuple
template<typename List>
struct to_tuple;

template<typename... Types>
struct to_tuple<type_list<Types...>> {
    using type = std::tuple<Types...>;
};
```

## 6. 实际应用案例

### 编译时状态机

```cpp
enum class State { Start, Processing, End };

template<State CurrentState, typename Event>
struct transition {
    static constexpr State next_state = CurrentState;
};

// 特化定义状态转换
template<>
struct transition<State::Start, struct StartEvent> {
    static constexpr State next_state = State::Processing;
};

template<>
struct transition<State::Processing, struct ProcessEvent> {
    static constexpr State next_state = State::Processing;
};

template<>
struct transition<State::Processing, struct EndEvent> {
    static constexpr State next_state = State::End;
};

template<State InitialState, typename... Events>
struct state_machine {
    static constexpr State final_state = process_events<InitialState, Events...>::state;
};

template<State CurrentState, typename Event, typename... RemainingEvents>
struct process_events {
    static constexpr State state = process_events
        transition<CurrentState, Event>::next_state, 
        RemainingEvents...
    >::state;
};

template<State CurrentState, typename Event>
struct process_events<CurrentState, Event> {
    static constexpr State state = transition<CurrentState, Event>::next_state;
};
```

### 编译时表达式求值

```cpp
// 抽象语法树节点
template<int Value>
struct Literal {
    static constexpr int eval() { return Value; }
};

template<typename Left, typename Right>
struct Add {
    static constexpr int eval() { return Left::eval() + Right::eval(); }
};

template<typename Left, typename Right>
struct Multiply {
    static constexpr int eval() { return Left::eval() * Right::eval(); }
};

// 使用：(2 + 3) * 4
using expr = Multiply<Add<Literal<2>, Literal<3>>, Literal<4>>;
constexpr int result = expr::eval();  // 20
```

### 编译时单位系统

```cpp
template<int M, int L, int T>  // Mass, Length, Time
struct Unit {
    static constexpr int mass = M;
    static constexpr int length = L;
    static constexpr int time = T;
};

template<typename Unit, double Value>
struct Quantity {
    static constexpr double value = Value;
    using unit = Unit;
};

// 基本单位
using Dimensionless = Unit<0, 0, 0>;
using Mass = Unit<1, 0, 0>;
using Length = Unit<0, 1, 0>;
using Time = Unit<0, 0, 1>;
using Velocity = Unit<0, 1, -1>;
using Acceleration = Unit<0, 1, -2>;

// 运算
template<typename U1, double V1, typename U2, double V2>
constexpr auto operator*(Quantity<U1, V1>, Quantity<U2, V2>) {
    using result_unit = Unit<U1::mass + U2::mass, 
                            U1::length + U2::length, 
                            U1::time + U2::time>;
    return Quantity<result_unit, V1 * V2>{};
}

// 使用
constexpr auto distance = Quantity<Length, 10.0>{};
constexpr auto time = Quantity<Time, 2.0>{};
constexpr auto velocity = distance * time;  // 类型检查会失败，应该是distance / time
```

## 7. 现代C++元编程 (C++17/20)

### if constexpr (C++17)

```cpp
template<typename T>
auto process(T value) {
    if constexpr (std::is_integral_v<T>) {
        return value * 2;
    } else if constexpr (std::is_floating_point_v<T>) {
        return value * 1.5;
    } else {
        return value;
    }
}
```

### 折叠表达式 (C++17)

```cpp
template<typename... Args>
auto sum(Args... args) {
    return (args + ...);  // 右折叠
}

template<typename... Args>
auto sum_left(Args... args) {
    return (... + args);  // 左折叠
}

template<typename... Args>
bool all_true(Args... args) {
    return (args && ...);
}

template<typename... Args>
void print_all(Args... args) {
    ((std::cout << args << " "), ...);
}
```

### Concepts (C++20)

```cpp
template<typename T>
concept Numeric = std::integral<T> || std::floating_point<T>;

template<typename T>
concept Printable = requires(T t) {
    std::cout << t;
};

template<Numeric T>
T multiply(T a, T b) {
    return a * b;
}

template<Printable T>
void print(const T& value) {
    std::cout << value << std::endl;
}
```

## 8. 性能考虑

### 编译时 vs 运行时

```cpp
// 编译时计算
template<int N>
struct CompileTimeFactorial {
    static constexpr int value = N * CompileTimeFactorial<N-1>::value;
};

template<>
struct CompileTimeFactorial<0> {
    static constexpr int value = 1;
};

// 运行时计算
int runtime_factorial(int n) {
    if (n <= 1) return 1;
    return n * runtime_factorial(n - 1);
}

// 使用
constexpr int ct_result = CompileTimeFactorial<10>::value;  // 编译时计算
int rt_result = runtime_factorial(10);  // 运行时计算
```

### 模板实例化控制

```cpp
// 显式实例化声明
extern template class std::vector<int>;
extern template class std::vector<double>;

// 显式实例化定义
template class std::vector<int>;
template class std::vector<double>;
```

## 最佳实践

1. **合理使用元编程**：不要过度使用，保持代码可读性
2. **优先使用标准库**：`<type_traits>`、`<utility>`等
3. **使用现代C++特性**：`if constexpr`、concepts等
4. **避免深层递归**：可能导致编译器栈溢出
5. **提供良好的错误信息**：使用`static_assert`和concepts
6. **测试编译时计算**：确保正确性

```cpp
// 良好的错误信息
template<typename T>
void safe_function(T value) {
    static_assert(std::is_arithmetic_v<T>, 
                  "T must be an arithmetic type");
    // 函数实现
}
```

C++元编程是一个强大的工具，能够创建高效、类型安全的代码，但需要谨慎使用以保持代码的可读性和可维护性。