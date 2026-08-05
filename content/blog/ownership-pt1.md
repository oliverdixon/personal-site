---
title: "Modelling Ownership in C++ (Part 1)"
date: 2026-08-04T11:47:34+01:00
draft: false
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

* [`T*`](https://en.cppreference.com/w/cpp/language/pointer) (the mutating raw pointer)
* [`const T*`](https://en.cppreference.com/w/cpp/language/pointer#Constness) (the observing raw pointer)
* [`T&`](https://en.cppreference.com/w/cpp/language/reference) (the mutating reference)
* [`const T&`](https://en.cppreference.com/w/cpp/language/reference#Lvalue_references) (the observing reference)
* [`T&&`](https://en.cppreference.com/w/cpp/language/reference#Rvalue_references) (the rvalue reference)
* [`const T&&`](https://en.cppreference.com/w/cpp/language/reference#Rvalue_references) (the constant rvalue reference)
* [`T&&`](https://en.cppreference.com/w/cpp/language/reference#Forwarding_references) (the forwarding reference)
* [`auto&&`](https://en.cppreference.com/w/cpp/language/reference#Forwarding_references) (the type-deducing forwarding
  reference)
* [`R (*)(Args...)`](https://en.cppreference.com/w/cpp/language/pointer#Pointers_to_functions) (the non-member function
  pointer)
* [`T C::*`](https://en.cppreference.com/w/cpp/language/pointer#Pointers_to_data_members) (the data member pointer)
* [`R (C::*)(Args...)`](https://en.cppreference.com/w/cpp/language/pointer#Pointers_to_member_functions) (the member
  function pointer)

A major milestone in a C++ programmer's journey is knowing when to use each of these. What are their strengths? What are
their weaknesses? Admittedly, few developers need to be intimately familiar with all of the constructs, since some of
them are just plain obscure, but having a working knowledge of the ideas behind them is required for leveraging them to
effectively write less buggy code.

To let you in on a secret: very few of these constructs are technically doing anything more than being a thin wrapper
around a raw pointer of the form `T*`. The common theme amongst these types is taking all the unadulterated power of the
`T*` and controlling access to allow programmers to express semantics within the syntax.

## Pointers are not an ownership model

If you take one thing away from this blog series, it should be this:

**Raw pointers are not obsolete: they're just useless for expressing semantics.**

Expressing semantics with syntax is really the guiding light for C++ language and standard library design: if you *say
what you mean* and *state your intentions*, then compliance can be verified statically, continuously, and anyone else
reading your code will build the mental model you intended. And in C-like languages, there is no aspect with more
nuanced semantics than the memory model:

* Who owns what?
* When is ownership transferred?
* What are the lifetime guarantees on this?
* Is this thread-safe?

To sharpen my earlier comment of "pointers do not cause bugs; programmers cause bugs": perhaps it would be more
diplomatic to say that raw pointers can allow bugs because they do not encode sufficient intent.

In that vein, I would like to provide a handful of tips on using the modern C++ to intentionally and confidently use
pointers in C++ to manage your in-memory objects, whether through the venerable `T*`, or through a modern abstraction. I
won't cover all of the above, but I'll provide a tour of the important subset. Importantly, I'll describe the semantic
properties that should be expressed in some typical circumstances, and how to express them using clear, unambiguous,
modern C++.

In this first post, I'll provide what are, in my opinion, the most crucial take-aways for immediately improving your
C++: questioning whether you need any pointers to represent your ownership model, and, if you do, what your default
should be.

## Prefer Values First

Before choosing any pointer-like type, ask whether the object can just exist directly in the place that owns it. In C++,
the cleanest ownership model is often not:

```c++
std::unique_ptr<Foo> foo;
```

but rather one of:

```c++
Foo foo;
std::optional<Foo> maybe_foo;
std::vector<Foo> foos;

class Container { Foo foo; };
Container container;
```

That is value ownership: the object is owned directly by its enclosing scope, class, or container. No heap allocation is
implied, no nullable state is implied, and no separate lifetime question is introduced. Destruction happens
automatically and predictably.

This is worth emphasising because `unique_ptr` is sometimes treated as the "modern C++ ownership type", but that is only
half true. `unique_ptr` models a particular kind of ownership: exclusive ownership through indirection. Ordinary values
also model ownership, and usually with less machinery.

For example, suppose a `Project` owns a collection of `Signal` objects. A first attempt might look like this:

```c++
class Project {
public:
    void add_signal(std::unique_ptr<Signal> signal)
    {
        signals.push_back(std::move(signal));
    }

private:
    std::vector<std::unique_ptr<Signal>> signals;
};
```

That may be the right design, but it should not be chosen merely because the `Project` "owns" the signals. If the
signals are just ordinary objects owned by the project, this is simpler:

```c++
class Project {
public:
    void add_signal(Signal signal)
    {
        signals.push_back(std::move(signal));
    }

private:
    std::vector<Signal> signals;
};
```

The second version says something very clear:

* A `Project` owns its `Signal` objects.
* A `Signal` does not have an independent lifetime outside the `Project`.
* There are no empty signal slots unless the container itself is empty.
* Destroying the `Project` destroys the signals.
* There is no heap allocation per signal merely for the sake of ownership.

That is a strong default. It is also easier to reason about, easier to test, and often better for locality. The `Signal`
objects live directly inside the vector, rather than being scattered around the heap behind a layer of pointers.

Of course, the `unique_ptr` version may still be justified. Indirection buys you something, and sometimes you really do
need the thing it buys. For example:

```c++
class Project {
private:
    std::vector<std::unique_ptr<Signal>> signals;
};
```

may be preferable when:

* `Signal` is a base class and the project stores different derived signal types.
* `Signal` is non-movable, or moving it would be awkward or expensive.
* Other objects need stable pointers or references to individual signals while the vector grows.
* Individual signals need to be transferred into or out of the project without moving the underlying object.
* The container needs nullable slots.
* The lifetime of a signal is deliberately decoupled from the storage layout of the container.

The important point is not that one version is universally better. The important point is that the pointer version says
more than "this object is owned". It says that ownership is indirect, nullable, transferable, and heap-allocated. If
those are not properties you actually need, then the pointer is adding complexity rather than expressing useful
semantics.

### Takeaway

The rough order of preference is:

```c++
Foo foo;                 // own one Foo directly
std::optional<Foo> foo;  // maybe own one Foo directly
std::vector<Foo> foos;   // own many Foo objects directly
std::unique_ptr<Foo> p;  // own one Foo indirectly
```

The rule is not "avoid pointers". The rule is: do not reach for indirection until indirection is buying you something.
Values should be the starting point because they express the simplest and strongest ownership relationship C++ has.

## Use `unique_ptr` for Exclusive Dynamic Ownership

I can say with confidence that `unique_ptr` is amongst the most useful, beautiful, and flexible symbols in the `std`
namespace. Introduced in C++11, the simplicity of its premise is difficult to overstate: wrap a raw pointer in a
container with a `=delete`'d copy constructor. It is so simple that I can include a simple implementation inline:

```c++
#include <utility>
#include <cassert>

template <typename T>
class unique_ptr {
public:
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

    T* operator->() const noexcept
    {
        assert(ptr != nullptr);
        return ptr;
    }
    
    T& operator*() const noexcept
    {
        assert(ptr != nullptr);
        return *ptr;
    }

private:
    T* ptr;
};
```

Alright, maybe that's not a *standard-compliant* implementation, but it is *a representative* implementation.
The [real deal](https://eel.is/c++draft/unique.ptr) provides support for custom deleters, conversions, and variably
sized objects.

Note the following crucial aspects:

* The copy constructor and copy-assignment operator are explicitly disabled;
* The move constructor and move-assignment operator are explicitly enabled.
* The stored raw pointer `T*` is available on the public API.

Despite removing functionality from the `T*` that it holds, `unique_ptr` allows programmers to write remarkably
expressive code. Let's connect each of the features shown in the sample implementation to the concrete semantics which
they are intended to convey:

| Feature                                              | Name                      | Meaning                                                                                                  |
|------------------------------------------------------|---------------------------|----------------------------------------------------------------------------------------------------------|
| `unique_ptr::unique_ptr(T*)`                         | Constructor               | Constructs a `unique_ptr` that assumes exclusive ownership of the supplied dynamically allocated object. |
| `unique_ptr::unique_ptr(const unique_ptr&) = delete` | Deleted copy constructor  | There cannot be more than one owner.                                                                     |
| `unique_ptr::unique_ptr(unique_ptr&&)`               | Enabled move constructor  | Ownership can be transferred, but only through an explicit move.                                         |
| `unique_ptr::~unique_ptr()`                          | Destructor                | The `unique_ptr` uses RAII: the lifetime of the stored object is tied to the lifetime of the unique_ptr. |
| `unique_ptr::release()`                              | Release member function   | Relinquishes ownership without deleting the object; the caller must arrange a new owner or deletion.     |
| `unique_ptr::get()`                                  | Observing member function | Allows the stored object to be viewed as a raw pointer.                                                  |

### Sources, Sinks, and Observers

One useful way to think about ownership is to classify functions by what they do with an object.

#### Sources

A source creates or returns ownership:

```c++
std::unique_ptr<Signal> load_signal(const std::filesystem::path& path)
{
    auto signal = std::make_unique<Signal>();

    // Populate the signal from disk...

    return signal;
}
```

The caller now owns the returned `Signal`:

```c++
std::unique_ptr<Signal> signal = load_signal("voice.wav");
```

#### Sinks

A sink accepts ownership. The clearest way to express that is to take a `std::unique_ptr<T>` by value:

```c++
class Project {
public:
    void add_signal(std::unique_ptr<Signal> signal)
    {
        signals.push_back(std::move(signal));
    }

private:
    std::vector<std::unique_ptr<Signal>> signals;
};
```

Calling the sink requires an explicit move:

```c++
auto signal = load_signal("voice.wav");

project.add_signal(std::move(signal));

// signal is now empty
```

That `std::move` is important. It makes the ownership transfer visible at the call site. The object has not been copied,
so there remains only one owner. Ownership has moved from the local variable into the `Project`.

#### Observers

An observer does not take ownership or participate in the ownership model whatsoever. If the object must exist, use a
reference:

```c++
void print_signal_summary(const Signal& signal)
{
    std::cout << signal.get_sample_rate() << '\n';
    std::cout << signal.get_sample_count() << '\n';
}
```

This can be called through the `unique_ptr` without transferring ownership:

```c++
auto signal = load_signal("voice.wav");
print_signal_summary(*signal);
// signal still owns the object
```

If the observer is allowed to receive no object, use a raw pointer:

```c++
void select_signal(Signal* signal)
{
    if (signal == nullptr) {
        clear_selection();
        return;
    }

    show_signal(*signal);
}
```

Again, no ownership is transferred:

```c++
select_signal(signal.get());
```

This is the important distinction:

```c++
void observe_signal(const Signal& signal);               // borrows, cannot be null
void maybe_observe_signal(const Signal* signal);         // borrows, may be null
void consume_signal(std::unique_ptr<Signal> signal);     // takes ownership
std::unique_ptr<Signal> make_signal();                   // returns ownership
```

In other words:

* return `unique_ptr<T>` from ownership-producing functions;
* pass `unique_ptr<T>` by value into ownership-consuming functions;
* pass `T&`, `const T&`, `T*`, or `const T*` to functions that merely observe.

### Takeaway

Use `std::unique_ptr<Foo>` to state that this scope has exclusive ownership of a dynamically allocated `Foo`, and that
any transfer of ownership must be explicit.

When the object really does need an independent lifetime, polymorphic storage, delayed construction, transfer between
owners, or stable address semantics, `std::unique_ptr<T>` is usually the right default.
