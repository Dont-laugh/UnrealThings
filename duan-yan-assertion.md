---
description: Unreal 里的断言并不是直接命名为 Assert，而是精细化分为 Check, Verify 和 Ensure 三类。
---

# 🙏 断言 (Assertion)

官方文档：

{% embed url="https://docs.unrealengine.com/5.2/en-US/asserts-in-unreal-engine/" %}

## 一、Check

### 1.1 check

check 与 assert 的定义最为接近，即表达式结果为 false 时，输出提示并中断程序。

在 Core\Public\Misc\AssertionMacros.h 文件中可找到 check 宏定义，其中宏 `DO_CHECK` 在 Development 和 Debug 配置中为 1，Test 和 Shipping 中等于 `USE_CHECKS_IN_SHIPPING` 。

```cpp
#if DO_CHECK
#ifndef checkCode
    #define checkCode(Code)      do { Code; } while ( false );
#endif
#ifndef verify
    #define verify(expr)         UE_CHECK_IMPL(expr)
#endif
#ifndef check
    #define check(expr)          UE_CHECK_IMPL(expr)
#endif

#define UE_CHECK_IMPL(expr) \
{ \
    if(UNLIKELY(!(expr))) \
    { \
        struct Impl \
        { \
            static void FORCENOINLINE UE_DEBUG_SECTION ExecCheckImplInternal() \
            { \
                FDebug::CheckVerifyFailedImpl(#expr, __FILE__, __LINE__, PLATFORM_RETURN_ADDRESS(), TEXT("")); \
            } \
        }; \
    Impl::ExecCheckImplInternal(); \
    PLATFORM_BREAK_IF_DESIRED(); \
    CA_ASSUME(false); \
    } \
}

#if !UE_BUILD_SHIPPING
    extern CORE_API bool GIgnoreDebugger;
    // This is named with PLATFORM_ prefix because UE_DEBUG_BREAK* are conditional on a debugger being detected, and PLATFORM_BREAK isn't
    #define PLATFORM_BREAK_IF_DESIRED() if (LIKELY(!GIgnoreDebugger)) { PLATFORM_BREAK(); }
#else
    #define PLATFORM_BREAK_IF_DESIRED() PLATFORM_BREAK();
#endif // !UE_BUILD_SHIPPING

#define PLATFORM_BREAK() (__nop(), __debugbreak())
```

### 1.2 checkSlow

checkSlow 只在开启了 `DO_GUARD_SLOW` 宏时与 check 行为一致，而 `DO_GUARD_SLOW` 只在 Debug 配置下才是 1。

`CA_ASSUME` 宏通常情况下为空，所以 Debug 配置下 checkSlow 与 check 等效，其余配置 checkSlow 无效。

```cpp
#if DO_GUARD_SLOW
    #define checkSlow(expr)               check(expr)
    #define checkfSlow(expr, format, ...) checkf(expr, format, ##__VA_ARGS__)
    #define verifySlow(expr)              check(expr)
#else
    #define checkSlow(expr)               { CA_ASSUME(expr); }
    #define checkfSlow(expr, format, ...) { CA_ASSUME(expr); }
    #define verifySlow(expr)              { if(UNLIKELY(!(expr))) { CA_ASSUME(false); } }
#endif

#if USING_CODE_ANALYSIS
    // ……
#else
    // Just to be safe, define all of the code analysis macros to empty macros
    #define CA_SUPPRESS( WarningNumber )
    #define CA_ASSUME( Expr )
    #define CA_CONSTANT_IF(Condition) if (Condition)
#endif
```

在 Core\Public\Windows\WindowsPlatformCodeAnalysis.h 中能找到 `CA_ASSUME` 的定义，所以在开启了 `USING_CODE_ANALYSIS` 的条件下，`CA_ASSUME` 才会借助编译器的代码分析能力来给出警告。

```cpp
#if USING_CODE_ANALYSIS
    #if !defined(__clang_analyzer__)
    // ……
    #define CA_ASSUME( Expr ) __analysis_assume( !!( Expr ) )
    #else
    // ……
    __declspec(dllimport, noreturn) void CA_AssumeNoReturn();
    #define CA_ASSUME( Expr )  (__builtin_expect(!bool(Expr), 0) ? CA_AssumeNoReturn() : (void)0)
    #endif
#endif
```

### 1.3 checkf/checkfSlow

作用与 check/checkSlow 一致，但提供格式化输出提示。

```cpp
/**
 * verifyf, checkf: Same as verify, check but with printf style additional parameters
 * Read about __VA_ARGS__ (variadic macros) on http://gcc.gnu.org/onlinedocs/gcc-3.4.4/cpp.pdf.
 */
#ifndef verifyf
    #define verifyf(expr, format,  ...) UE_CHECK_F_IMPL(expr, format, ##__VA_ARGS__)
#endif
#ifndef checkf
    #define checkf(expr, format,  ...)	UE_CHECK_F_IMPL(expr, format, ##__VA_ARGS__)
#endif

#define UE_CHECK_F_IMPL(expr, format, ...) \
    { \
        if(UNLIKELY(!(expr))) \
        { \
            DispatchCheckVerify([] (const auto& LFormat, const auto&... UE_LOG_Args) UE_DEBUG_SECTION \
            { \
                FDebug::CheckVerifyFailedImpl(#expr, __FILE__, __LINE__, PLATFORM_RETURN_ADDRESS(), LFormat, UE_LOG_Args...); \
            }, format, ##__VA_ARGS__); \
            PLATFORM_BREAK_IF_DESIRED(); \
            CA_ASSUME(false); \
        } \
    }
```

### 1.4 checkCode

checkCode 宏只是在一次性的 do-while 循环中执行一遍表达式。所以 checkCode 的作用是在 Debug 和 Development 运行一段代码，而在未开启 `USE_CHECKS_IN_SHIPPING` 的 Test 和 Shipping 不运行。

```cpp
#ifndef checkCode
    #define checkCode(Code)      do { Code; } while ( false );
#endif
```

### 1.5 checkNoEntry

checkNoEntry 在运行这一行时就会中断程序。

```cpp
/**
 * Denotes code paths that should never be reached.
 */
#ifndef checkNoEntry
    #define checkNoEntry()    check(!"Enclosing block should never be called")
#endif
```

### 1.6 checkNoReentry

checkNoReentry 在第二次运行这一行时会中断程序。

```cpp
/**
 * Denotes code paths that should not be executed more than once.
 */
#ifndef checkNoReentry
    #define checkNoReentry() { static bool s_beenHere##__LINE__ = false;                                       \
                               check( !"Enclosing block was called more than once" || !s_beenHere##__LINE__ ); \
                               s_beenHere##__LINE__ = true; }
```

### 1.7 checkNoRecursion

checkNoRecursion 在递归运行到这一行时会中断程序。

```cpp
class FRecursionScopeMarker
{
public: 
    FRecursionScopeMarker(uint16 &InCounter) : Counter( InCounter ) { ++Counter; }
    ~FRecursionScopeMarker() { --Counter; }
private:
    uint16& Counter;
};

/**
 * Denotes code paths that should never be called recursively.
 */
#ifndef checkNoRecursion
    #define checkNoRecursion()  static uint16 RecursionCounter##__LINE__ = 0;                                           \
	                        check( !"Enclosing block was entered recursively" || RecursionCounter##__LINE__ == 0 ); \
	                        const FRecursionScopeMarker ScopeMarker##__LINE__( RecursionCounter##__LINE__ )
#endif
```

### 1.8 unimplemented

unimplemented 和 checkNoEntry 作用相同，都是运行即中断，但提示信息不同，用在不应该调用的虚函数中。

```cpp
#ifndef unimplemented
    #define unimplemented()    check(!"Unimplemented function called")
#endif
```



## 二、Verify

### 2.1 verify/verifyf

从引擎代码来看，verify/verifyf 和 check/checkf 完全一致：

```cpp
#if DO_CHECK
#ifndef checkCode
    #define checkCode( Code )    do { Code; } while ( false );
#endif
#ifndef verify
    #define verify(expr)         UE_CHECK_IMPL(expr)
#endif
#ifndef check
    #define check(expr)          UE_CHECK_IMPL(expr)
#endif

#ifndef verifyf
    #define verifyf(expr, format,  ...) UE_CHECK_F_IMPL(expr, format, ##__VA_ARGS__)
#endif
#ifndef checkf
    #define checkf(expr, format,  ...)	UE_CHECK_F_IMPL(expr, format, ##__VA_ARGS__)
#endif
```

### 2.2 verifySlow

verifySlow 在 shipping 配置下，会执行表达式但忽略返回值；在 Debug 配置下，与 checkSlow 行为一致。

```cpp
//
// Check for development only.
//
#if DO_GUARD_SLOW
    #define checkSlow(expr)               check(expr)
    #define checkfSlow(expr, format, ...) checkf(expr, format, ##__VA_ARGS__)
    #define verifySlow(expr)              check(expr)
#else
    #define checkSlow(expr)               { CA_ASSUME(expr); }
    #define checkfSlow(expr, format, ...) { CA_ASSUME(expr); }
    #define verifySlow(expr)              { if(UNLIKELY(!(expr))) { CA_ASSUME(false); } }
#endif
```



## 三、Ensure

在 Debug 和 Development 中， `DO_ENSURE` 宏均为 1，而在 Test 和 Shipping 中等于 `USE_ENSURES_IN_SHIPPING`。

```cpp
#if DO_ENSURE && !USING_CODE_ANALYSIS // The Visual Studio 2013 analyzer doesn't understand these complex conditionals
    #define UE_ENSURE_IMPL(Capture, Always, InExpression, ...) \
        (LIKELY(!!(InExpression)) || (DispatchCheckVerify<bool>([Capture] () UE_DEBUG_SECTION \
        { \
            static bool bExecuted = false; \
            if ((!bExecuted || Always) && FPlatformMisc::IsEnsureAllowed()) \
            { \
                bExecuted = true; \
                FDebug::OptionallyLogFormattedEnsureMessageReturningFalse(true, #InExpression, __FILE__, __LINE__, PLATFORM_RETURN_ADDRESS(), ##__VA_ARGS__); \
                if (!FPlatformMisc::IsDebuggerPresent()) \
                { \
                    FPlatformMisc::PromptForRemoteDebugging(true); \
                    return false; \
                } \
                return true; \
            } \
            return false; \
        }) && ([] () { PLATFORM_BREAK_IF_DESIRED(); } (), false)))

    #define ensure(           InExpression                ) UE_ENSURE_IMPL( , false, InExpression, TEXT(""))
    #define ensureMsgf(       InExpression, InFormat, ... ) UE_ENSURE_IMPL(&, false, InExpression, InFormat, ##__VA_ARGS__)
    #define ensureAlways(     InExpression                ) UE_ENSURE_IMPL( , true,  InExpression, TEXT(""))
    #define ensureAlwaysMsgf( InExpression, InFormat, ... ) UE_ENSURE_IMPL(&, true,  InExpression, InFormat, ##__VA_ARGS__)

#else	// DO_ENSURE

    #define ensure(           InExpression                ) (!!(InExpression))
    #define ensureMsgf(       InExpression, InFormat, ... ) (!!(InExpression))
    #define ensureAlways(     InExpression                ) (!!(InExpression))
    #define ensureAlwaysMsgf( InExpression, InFormat, ... ) (!!(InExpression))

#endif	// DO_CHECK
```

### 3.1 ensure

ensure 只会执行表达式一次，并且不会中断程序的运行，所以用于非致命的断言。当 `DO_ENSURE` 为 0 时，总会执行表达式，不会输出提示。

### 3.2 ensureMsgf

是 ensure 的带格式化输出的版本。

### 3.3 ensureAlways

与 ensure 只有一点区别：每次都会执行表达式。

### 3.4 ensureAlwaysMsgf

是 ensureAlways 的带格式化输出的版本。

