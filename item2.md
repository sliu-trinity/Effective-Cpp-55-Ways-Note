# Item 2: Prefer consts, enums, and inlines to #defines.

## I. Problem with #define
Prefer the compiler to the preprocessor because **#define** may be treated as if it;s not part of the language per se. 

For instance:  
```C++
#define ASPECT_RATIO 1.653
```
the symbolic name ASPECT_RATIO may never be seen by compilers. It might be removed by preprocessor before the source code ever gets to a compiler. As a result, it may not get entered into the source table, which can be confusing if you get an error during compilation involving the use of the constant, because the error message may refer to the **value**(1.653) rather than the name ASPECT_RATIO. This is especially confusing when defined in a header file you did't write. Not only you have no idea where that comes from, it's extremely time-confusing to debug. This problem can also crop up in a symbolic debugger because the name may not be in the symbol table.

A nice alternative is to replace the macro with a constant: 
```C++
const double AspectRatio = 1.653;    //uppercase names are usually for macros, hence the name change
```
**Advantages**: 
1. A language constant like this is ensured to be seen by compilers and entered to symbol tables. 
2. In the case of a floating point constant, use of constant may yield smaller code than using **#define**. **With macro**, **the preprocessor's blind substitution** of the macro name ASPECT_RATIO with 1.653 could result in **multiple copies** of 1.653 in your code. **With the use of constant, only 1 copy will be generated**.

Two special cases when replacing **#define**:
1. **Defining constant pointers**: because constant definitions are typically put in header file, where many different source files will include them, pointer should be declared const in addition to the value it points to. 
Ex: to define a constant char*-based string in header file
```C++
const char * const authorName = "Scott Meyers";
//or 
const std::string authorName("Scott Meyers"); //generally prefered compare to the char* based progenitors
```

2. **Make class-specific constants 'static'** this is to **limit the scope** of a constant to a class, and make it a member and ensure there is **at most one copy** of the constant. Ex: 
```C++ 
class GamePlayer {
private: 
  static const int NumTurns = 5;   //constant declaration
  int scores[NumTurns];            //use of constant
  ...
}
```
Note: 
  * This ia *declaration* not a *definition*. Usually C++ requires you to provide definition for anything you use, but class-specific constants that are static and of integral types(integers, chars, bools) are an exception. If you want to take address of it(which you can't do with just a declaration) or compiler incorrectly insists on a definition, you can provide a separate definition like: 
  ```C++
  const int GamePlayer::NumTurns;  //definition of NumTurns
  ```
  Initial value is not permitted at the point of definition, because the initial value of class constants are provided where the constant is declared. Thus this code has to be put in an implementation file, not a header file. 
  
  * **#define** cannot be used to create a class-specific constant, since it don't respect scope. Once a macro is defined, it's in force for the rest of the compilation unless there is a undefine somewhere along the line. 
  * There is also no encapsulation for macros like **#define**, i.e., there is no such thing as a "private" **#define**.
  * In-class initialization is only allowed for integral types and constants. In cases where the aboved syntax cannot be used, put initial value at the point of definition: 
  ```C++
  class CostEstimate {
  private: 
    static const double FudgeFactor;  //declaration of static class constant; goes in header file
    ...
  }
  const double CostEstimate::FudgeFactor=1.35 //definition of static class constant; goes to impl. file
  ```
  
  ## II."the enum hack" 
  used when you need the value of a class constant during compilation of the class. This takes advantage of the fact that the values of an enumerated type can be used where ints are expected: 
```C++
class GamePlayer {
private: 
  enum {NumTurns=5};  //makes NumTurns a symbolic name for 5
  int scores[NumTurns]; //fine
  ...
}
```
Advantages: 
1. enum hack behaves more like #define than a const does, which is sometimes you want. It's legal to take the address of a const but not to take the address of a enum (typically also #define). If you dont want to let people get a pointer or reference to some integral constants, enum can enforce that constraint.
2. Some sloppy conpilers may set aside storage for const objects of integral types unless you create a pointer or reference to the object. Enum can prevent this, since like #define, it never results in that kind of unnecessary memory allocation. 
3. It's pragmatic. A lot of people use it so one should recognize it when seeing it. It's a fundamental technique of*template metaprogramming*.

## III. Problem with function-like macros
```C++
//call f with the maximum of a and b
#define CALL_WITH_MAX(a,b) f((a)>(b) ? (a):(b))
```
Some people use macros like this to reduce overhead of function calls, but it has many painful drawbacks： 
1. **Redundancy**: number of parenthesis. Easy to make mistakes.
2. **Undefined behavior**: weird stuff can happen, even though you get all correctly: 
```C++
int a=5, b = 0;
CALL_WITH_MAX(++a, b); //a is incremented twice
CALL_WITH_MAX(++a, b+10); //a is incremented once
```

**Solution**: Using a template for an inline function that grants predictable behavior and type safety: 
```C++
template<typename T> 
inline void callWithMax(const T& a, const T&b) { //because we dont know what T is
  f(a>b ? a:b)                                   //We pass by reference to const
}
```
**Advantage**: 
1. Type safe - generates a whole family of functions, each of which takes two objects and return the greater
2. No undefined behavior - evaluating parameters multiple times 
3. No more parenthesis 
4. A real function: it obeys scope and access rules.

```diff
- Things to Remember
```
* For simple constants, prefer const objects or enums to #defines.
* For function-like macros, prefer inline functions to #defines.
