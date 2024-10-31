---
layout: math
title: Java Generics
date: 2022-12-1 11:23
post-link:
---

## Why using Generics?

Generics is something used to preserve type safety of Java when still trying to have polymorphism in our code. Without Java generics, we can still use co-variant array to achieve this, but subject to pitfalls.


```java
Integer[] ints = new Integer[10];
void foo(Number[] nums) { // this is one kind of polymorphism, subtype polymorphism
    nums.add(3.14);
}
foo(ints);
```

Well, we can fool the compiler, but cannot fool the runtime system. The above code will get a runtime failure.

When having generics, we do this:

```java
static<T> void foo(List<T> nums) {//this is parametric polymorphism = java generics  
    nums.add(3.14); // type incompatible: double cannot be converted to T  
}  
foo<Integer>(ints)  
foo<Float>(floats)
```


We achieve writing code polymorphismly, while still preserving the type safety.

And we cannot do this anymore:

```java
Object[] o = new Integer[10];  
o[1] = "str"; // fool the compiler again, but will get an arrayStoreException  
// change to collection type:  
ArrayList[Object] o = new ArrayList<Integer> ints; // compile failure, generics is invariant!
```

Using covariant array to achieve polymorphism can cause many bugs when the system starts running. We better take care of such bugs at compile time.

Well, that's partly why Java is considered as a strongly-typed language.

> A slight digression, `c/c++` is usally considered as weakly-typed language, because pointers can be casted arbitrarily, which makes the type less strict and cause memory issues. `char* ptr = reinterpret_cast<char*>(0xffff0000);` we are casting a memory address to char pointer!
> 
> In that sense, Java is not perfect in type safety: null breaks the type safety. A string variable can only be in string type? no, it can be null, so does the other types. well, this can cause many runtime failures, while we are expecting a string variable but a null variable. Kotlin solves this issue by having a nullable type system, and is pretty elegantly designed.

## Is java generics perfect?

Java generics, most of the cases are non-refiable. That means the type information is erased (due to type erasure) at runtime. occationally, this makes the exception floating around the system, make the bug hard to track.

because of that, it's recommended not use parameterized varargs. Here's an example how non-refiable varargs behaves differently than refiable varargs.

```java
public static void faultyMethod(List<String>... l) {  
    Object[] objectArray = l;           // Valid  
    objectArray[0] = Arrays.asList(42); // Also valid. fool the runtime. at runtime objectarray[0] is a List (type information removed), and the rhs is also a List. The exception will not be thrown here, possibly thrown much later, which causes hard problems for debugging.  
}

public static void faultyMethod(String... l) {  
    Object[] objectArray = l; // Valid  
    objectArray[0] = 42; // ArrayStoreException thrown here. at runtime, objectarray has string type, but passed in a integer  
}
```

> [https://stackoverflow.com/questions/32291515/varargs-heap-pollution-whats-the-big-deal](https://stackoverflow.com/questions/32291515/varargs-heap-pollution-whats-the-big-deal)

similarly: Arrays of concrete paramaterized types are inherently broken

unchecked cast can also cause similar problem, the `classcastexception` is thrown, but sometimes could be too late.

#### what's unchecked cast warning: 
Simply put, **we’ll see this warning when casting a raw type to a parameterized type without type checking**.

```java
public static void main(String[] args){  
    HashMap<String, Integer> mp = (HashMap<String, Integer>) foo(); // unchecked cast warning  
    mp.get("andrew"); // okay  
    mp.get("alan"); // okay  
    // ..... multiple calls.  
    Integer s = mp.get("alan"); // classcastexception  

}  

static HashMap foo() {  
    HashMap mp = new HashMap();  
    mp.put("andrew", 25);  
    mp.put("alan", "funk");  
    return mp;  
}
```

As such, some argues for refiable generics.

## How does type erasure work?

after compilation, all type variables are gone. A question has to be asked, can we have refiable generics and not do type erasure? We can but Java won't, one of the reason is backward-compatibility

```java
class Pair<K, V> {  
    K key;  
    V value;  
    static void foo() {  
        // before compilation  
        Pair<Integer, String> p = new Pair<Integer, String>();  
        String s = p.value;  
        // after compilation:  
        Pair p = new Pair();  
        String s = (String) p.value;  
    }  
}
```

The advantage is there's no runtime overhead and is backward compatability with older versions of non-generic java.

### How to use Generics?

1. no upper bound and lower bound
    ```java
    List<T> f;  
    static<T> int foo(T f) {...}
    ```
    `f` is invariant in such way. meaning `List<Integer>` is not subtype of `List<Object>`

2. upper bounded (producer extends), co-variant

3. lower bounded (consumer super), contra-variant

```java
static<T> void copy(List<? extends T> src, List<? super T> dest) {  
    for (int i = 0; i < src.size(); i++) {
        T t = src.get(i); // producer extends  
        dest.add(t); // consumer super  
    }  
    return dest;  
}
// not right syntax but you get the point  
// by using wildcard we can copy a list of integers to a list of numbers  
// without wildcard, just invariant collection type, even it's safe to do so, compiler gives us the error, which is frustrating.  
utils.<Number>copy(List<Integer> o, List<Number> s)
```

### What is polymorphism?

Polymorphism is a kind of abstraction in Programming Languages

Procedure, function, method is the abstraction of computation; ADT(Abstract Data Types) is the abstraction of data; Polymorphism is the abstraction over type.

**Three kinds of polymorphism:**

1. ad-hoc polymorphism: function overloading

2. subtype polymorphism

3. parameterized polymorphism