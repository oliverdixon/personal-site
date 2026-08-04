---
title: "Pointers in C++"
date: 2026-08-04T11:47:34+01:00
draft: true
---

## Motivation

Who doesn't love pointers? Maybe students learning about them for the first time have some reservations, but on balance,
I'd say that they're fairly popular.

The humble asterisk (or [`^`](https://www.freepascal.org/docs-html/ref/refse15.html), or [
`'Access`](https://learn.adacore.com/courses/intro-to-ada/chapters/access_types.html))
gives us near-unrestricted access to a system's memory and opens access to new classes of programming paradigms. The
concept of uniquely identifying a chunk of information using only a lightweight 32-,
[48-](https://en.wikipedia.org/wiki/Burroughs_Large_Systems), or 64-bit number is severely underrated, and one of my
major gripes with some higher-level languages is that they attempt to hide this glorious abstraction because *somebody
else* has decided that *you* cannot be trusted to wield their power responsibly. But, with enough syntax, most languages
still do support pointers as a fundamental mechanism to programming computers:

| Language | Pointer Declaration                                            |
|----------|----------------------------------------------------------------|
| C/C++    | `int *p = &x;`                                                 |
| BCPL     | `LET p = lv x; rv p := 42`                                     |
| Pascal   | `var p: ^Integer; p := @x;`                                    |
| Fortran  | `integer, target :: x; integer, pointer :: p; p => x`          |
| Ada      | `type Int_Ptr is access all Integer; P : Int_Ptr := X'Access;` |
| Go       | `var p *int = &x`                                              |
| Rust     | `let p: &i32 = &x;`                                            |
| Zig      | `const p: *i32 = &x;`                                          |
| Swift    | `let p = UnsafePointer(&x)`                                    |
| Java     | `int[] a = {1, 2, 3}; int[] r = a;`                            |

Sure, not all of the above are pointers in the C sense: some are raw addresses, some are checked references, some are
GC-managed object references, and some are non-owning views or borrows. But they all express the same broad idea: one
value can refer to storage, or an object that lives somewhere else, *without owning it*.

As languages have evolved, there have been repeated attempts to move away from manipulation of raw pointers because
they're "unsafe", "cause bugs", and represent historical technical debt inherited from C. I don't want to
[get too philosophical](https://philosophy.stackexchange.com/q/10975), but I would argue that *pointers don't cause
bugs; programmers cause bugs*. And most of those bugs arise as a result of a C-era misunderstanding of the purpose of
pointers. With a focus on C++, I would like to take the admittedly high risk of arguing that pointers are not bug prone,
and that rather than replacing the utility of pointers, modern abstractions only enhance the expressiveness of the
humble `T*`.

## The C++ Situation

The C++ standard and technical specifications define a worryingly large number of symbols with very similar names in the
`std` namespace. And, on the surface, they all seem to do the same thing: store a memory address.

* [`unique_ptr<T>`](https://en.cppreference.com/w/cpp/memory/unique_ptr)
* [`shared_ptr<T>`](https://en.cppreference.com/w/cpp/memory/shared_ptr)
* [`weak_ptr<T>`](https://en.cppreference.com/w/cpp/memory/weak_ptr)
* [`auto_ptr<T>`](https://en.cppreference.com/w/cpp/memory/auto_ptr) (removed in C++17)
* [`reference_wrapper<T>`](https://en.cppreference.com/w/cpp/utility/functional/reference_wrapper)
* [`atomic<T*>`](https://en.cppreference.com/w/cpp/atomic/atomic)
* [`atomic<shared_ptr<T>>`](https://en.cppreference.com/w/cpp/memory/shared_ptr/atomic2)
* [`atomic<weak_ptr<T>>`](https://en.cppreference.com/w/cpp/memory/weak_ptr/atomic2)
* [`out_ptr_t`](https://en.cppreference.com/w/cpp/memory/out_ptr_t)
* [`inout_ptr_t`](https://en.cppreference.com/w/cpp/memory/inout_ptr_t)
* [`indirect<T>`](https://en.cppreference.com/w/cpp/memory/indirect)
* [`polymorphic<T>`](https://en.cppreference.com/w/cpp/memory/polymorphic)
* [`hazard_pointer`](https://en.cppreference.com/w/cpp/header/hazard_pointer)
* [`span<T>`](https://en.cppreference.com/w/cpp/container/span)
* [`mdspan<T, Extents, LayoutPolicy, AccessorPolicy>`](https://en.cppreference.com/w/cpp/container/mdspan)
* [`basic_string_view<CharT>`](https://en.cppreference.com/w/cpp/string/basic_string_view)
* [`function_ref<Sig>`](https://en.cppreference.com/w/cpp/utility/functional/function_ref)
* [`experimental::observer_ptr<T>`](https://en.cppreference.com/w/cpp/experimental/observer_ptr)
* [`experimental::propagate_const<T>`](https://en.cppreference.com/w/cpp/experimental/propagate_const)
* [`experimental::atomic_shared_ptr<T>`](https://en.cppreference.com/w/cpp/experimental/atomic_shared_ptr)
* [`experimental::atomic_weak_ptr<T>`](https://en.cppreference.com/w/cpp/experimental/atomic_weak_ptr)

If that wasn't enough, the core language also defines (and inherits from C) a number of notational presents that
apparently do the same:

* [`T*`](https://en.cppreference.com/w/cpp/language/pointer)
* [`const T*`](https://en.cppreference.com/w/cpp/language/pointer#Constness)
* [`T&`](https://en.cppreference.com/w/cpp/language/reference)
* [`const T&`](https://en.cppreference.com/w/cpp/language/reference#Lvalue_references)
* [`T&&`](https://en.cppreference.com/w/cpp/language/reference#Rvalue_references)
* [`const T&&`](https://en.cppreference.com/w/cpp/language/reference#Rvalue_references)
* [`T&&` forwarding reference](https://en.cppreference.com/w/cpp/language/reference#Forwarding_references)
* [`auto&&` forwarding reference](https://en.cppreference.com/w/cpp/language/reference#Forwarding_references)
* [`R (*)(Args...)`](https://en.cppreference.com/w/cpp/language/pointer#Pointers_to_functions)
* [`T C::*`](https://en.cppreference.com/w/cpp/language/pointer#Pointers_to_data_members)
* [`R (C::*)(Args...)`](https://en.cppreference.com/w/cpp/language/pointer#Pointers_to_member_functions)

A major milestone in a C++ programmer's journey is knowing when to use each of these. What are their strengths? What are
their weaknesses? Admittedly, few developers need to be intimately familiar with all of the constructs, since some of
them are just plain obscure, but having a working knowledge of the ideas behind them is required for leveraging them to
effectively write less buggy code.

To let you in on a secret: very few of these constructs are technically doing anything more than being a thin wrapper
around a raw pointer of the form `T*`. The common theme amongst these types is taking all the unadulterated power of the
`T*` and controlling access to allow programmers to express semantics within the syntax.

If you take one thing away from this post, it should be this:

**Pointers are not obsolete: they're just useless for expressing semantics.**

Expressing semantics with syntax is really the guiding light for C++ language and standard library design: if you *say
what you mean* and *state your intentions*, then compliance can be verified statically, continuously, and anyone else
reading your code will build the mental model you intended. And in C-like languages, there is no aspect with more
nuanced semantics than the memory model:

* Who owns what?
* When is ownership transferred?
* What are the lifetime guarantees on this?
* Is this thread-safe?

In that vein, I would like to provide a handful of tips on using the modern C++ to intentionally and confidently use
pointers in C++, whether through the venerable `T*`, or through a modern abstraction. I won't cover all of the above,
but I'll provide a tour of the important subset. Importantly, I'll describe the semantic properties should be expressed
in some typical circumstances, and how to express them using clear, unambiguous, modern C++.

## Tip #1: Prefer Values First

## Tip #2: Use `T*` for Nullable Non-Owning Observation

## Tip #3: Use `T&` for Required Non-Owning Access

## Tip #4: Use `unique_ptr` for Exclusive Dynamic Ownership

I can say with confidence that `unique_ptr` is amongst the most useful, beautiful, and flexible symbols in the `std`
namespace. Introduced in C++11, the simplicity of its premise is difficult to overstate: wrap a raw pointer in a
container with a `=delete`'d copy constructor. It is so simple that I can include its full implementation inline:

```c++
#include <utility>
#include <cassert>

template <typename T>
class unique_ptr {
public:
    constexpr unique_ptr() noexcept
        : ptr(nullptr)
    {
    }

    explicit unique_ptr(T* ptr) noexcept
        : ptr(ptr)
    {
    }

    ~unique_ptr()
    {
        delete ptr;
    }

    unique_ptr(const unique_ptr&) = delete;
    unique_ptr& operator=(const unique_ptr&) = delete;

    unique_ptr(unique_ptr&& other) noexcept
        : ptr(other.ptr)
    {
        other.ptr = nullptr;
    }

    unique_ptr& operator=(unique_ptr&& other) noexcept
    {
        if (this != &other) {
            delete ptr;

            ptr = other.ptr;
            other.ptr = nullptr;
        }

        return *this;
    }

    [[nodiscard]] T* get() const noexcept
    {
        return ptr;
    }

    [[nodiscard]] T* release() noexcept
    {
        T* old = ptr;
        ptr = nullptr;
        return old;
    }

    void reset(T* new_ptr = nullptr) noexcept
    {
        if (ptr != new_ptr) {
            delete ptr;
            ptr = new_ptr;
        }
    }

    void swap(unique_ptr& other) noexcept
    {
        std::swap(ptr, other.ptr);
    }

    T& operator*() const noexcept
    {
        assert(ptr != nullptr);
        return *ptr;
    }

    T* operator->() const noexcept
    {
        assert(ptr != nullptr);
        return ptr;
    }

    explicit operator bool() const noexcept
    {
        return ptr != nullptr;
    }

private:
    T* ptr;
};
```

Alright, maybe that's not a *standard-compliant* implementation, but it is *an* implementation.
The [real deal](https://eel.is/c++draft/unique.ptr) provides support for custom deleters, conversions, and variably
sized objects.

## Tip #5: Use `shared_ptr` Only for Shared Ownership

## Tip #6: Use `span`/`string_view` for Borrowed Ranges

## Tip #7: Use `weak_ptr` to Observe Shared Ownership Without Extending Lifetime

## Tip #8: Use `reference_wrapper` When You Need Reference Semantics in an Object

## The Cheatsheet
