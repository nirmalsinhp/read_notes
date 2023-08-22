# Type Deduction:
### function templates
    - expr is reference or pointers : reference/pointers is ignored, constness is not. than pattern match.
    - expr is universal reference(&&) : if expr is lvalue , param is lvalue, otherwise follow rule 1.
    - expr is value type - pass by value, ignores constness & reference both.
    - array & functions pointers are treated as pointer passed to function, so those rule applied.

### auto type deduction
    - auto type deduction similar to type deduction for function templates, where auto is considerd T, and declaration expression is like param, and initial value is expr
    const auto x = 12;  - 12 is expr, auto is T, const auto  is param type, x is const int.
    - initializer list does not work well with auto, deduced type for {12} will be initializer_list<int>
    - So the only real difference between auto and template type deduction is that auto assumes that a braced initializer represents a std::initializer_list, but template type deduction doesn’t.
    - auto in a function return type or a lambda parameter implies template type deduction, not auto type deduction.

### decltype deductions
    - provides info about type of expression or variable/name
    - cab be used for type deduction for function return value based on template params & operations
    - since c++14, function return type can be auto deduced, use auto as return type for that. but, these rules do not work well for function returning references, as during deduction refernces are ignored, use decltype(auto) to handle this.
    - auto type deduction is usually the same as template type deduction, but auto type deduction assumes that a braced initializer represents a std::initializer_list, and template type deduction doesn’t. 
    - auto in a function return type or a lambda parameter implies template type deduction, not auto type deduction.

### checking types
    - to check types deduced by compiler, can use __PRETTY_FUNCTION__ macro, declare class template but do not declare it, & try to instantiate it, boost type index.

# auto

### prefer auto to explicit type declarations
    - can not left uninitialized, so you cant forget to initialize it.
    - no need to write long name like typename iterator_traits<iter>::value_type.
    - some types like lambda(type erasure) are known only to compiler, use auto for it.
    - can use auto as a type in lambda function since c++14.
    - with c++20, can use auto as function parameter.
    - lambda can be stored in function, but it is slower than storing it in closure using auto.

### Use the explicitly typed initializer idiom when auto deduces undesired types.
    -  the explicitly typed initializer idiom.
    - when functions/expr returns invisible proxy objects, auto deduce that type, which are in most cases not usable, for example vector<bool>::operator[] returns __reference__ type, which is not bool, but implicitly convertable to bool.
    - in this cases, provide type explicitly or cast before assigning to auto using static_cast.

# moving to modern c++

### using { } vs ()
    -  3 types of initilization
        1. () - int x(2) 
        2. = -  int x = 42; same for PODs, for user defined copy ctor used.
        3. {} - inx x{43}; or int x = {42}; -- called uniform initialization, idea that all initilization can use braces. ints & containers alike.
    - () can't be used for default initilization of member variables;
    - =  can't be used for non copyable types- like atomic, 
    - {} works everywhere, so uniform initialization.
    - {} does not allow narrowing conversions.
    - most vexing parse does not affect {} initilizations.
    - BUT, {} initialization does not work well with auto, as types are considered initializer_list, and if there is ctor taking initializer list, it will be picked up over any other ctor, as long as values can be converted to it, gives surprizing results.

### prefer nullptr to 0 and NULL
    - nullptr can be assigned to any pointer, easily convertible to all raw pointers.
    - can not bind to smart pointer passed as lvalue ref.
    - decltype(auto)  deduces return type from c++14, trailing return type with auto can be used in c++11.
    - avoid overloading on integral & pointer types, 

### prefer alias declarations to typedefs.
    - typedefs : typedef vector<int> vi;
    - using : using vi = vector<int>;
    - only using works to define alias templates.
    '''
    template <typename T>
    using vt = vector<T>;
    '''
    - doing this with typedefs is tricky and cumbersome, with typedefs using typename is needed when using that aliases, alias template works just using alias.

### Prefer scoped enums to unscoped enums.
    -  enum color {red, black}; : unscoped enums are visible to outer scopes. convertible to integrals.
    -   enum class color(red, black); : scoped enums(enum classes) are limited to enum class scope, so same name can be used in outer scope with diff variables & enum classes. are not implicitly convertible to integrals. use static cast to convert.
    - enum class can be forward declared.
    - default underlying type of enum is int, to override enum class color : char; 


###  Prefer deleted functions to private undefined ones.
    - for special member functions that are autogenerated.
    - we mostly need functions like copy ctors & copy = operators to be unavailable in some cases, like streams for example.
    - C++98 - declare them private & do not define them. 
    - since C++11 these functions are declared public & deleted. i.e A(const A&) = delete.
    - any function can be deleted, not just special/member functions.
    - deleted functions take part in overload resolution, and throws an error if it is a best match.
    - delete works with template specializations also, private does not as you can not provide specialization and base template diff access specifiers.

### Declare overriding functions override.
    - for virtual functions to be overridden, many requirements must be met as follows
        - The base class function must be virtual.
        - The base and derived function names must be identical (except in the case of destructors).
        - The parameter types of the base and derived functions must be identical.
        - The constness of the base and derived functions must be identical.
        - The return types and exception specifications of the base and derived functions must be compatible.
        - with C++11, functions’ reference qualifiers must be identical. 
            - reference qualifier make member functions works with lvalues or rvalues only, suffix & and && respectively used.
    - using override explicitly in derived, throws compiler error, if function is not actually an override. or does not follow any of the above pre-requisits.
    
### Prefer const_iterators to iterators.
    - in c++98 using const iterators were troublesome, you can only get const iterators from const containers, even then using const iterators in some algos, did not work, and there is no portable conversion from iterator to const iterators.
    - c++11 cbeing factory method, gives const iterators for non-const containers also, and const iterators works everywhere even in insert, erase as thet are just providing positions.
    - calling begin on const containers return const iterators

###  Declare functions noexcept if they won’t emit exceptions.
    - specify noexcept if function does not throw/emit any exceptions.
    - it can improve performace for std vector push_back, emplace back, when there is need for resize, all current data can be moved, but for strong exception safe gurantee it needs to know if moves won't throw.
    - Exception-neutral functions are never noexcept
    - Move & swaps should be noexcept
    - By default, all memory deallocation functions and all destructors—both user-defined and compiler-generated—are implicitly noexcept.

### Use constexpr whenever possible.
    - const is not same as constexpr, const values does not need to be known at compile time.
    - constexpr objects are const and are initialized with values known during compilation.
    - constexpr functions can produce compile-time results when called with arguments whose values are known during compilation.
    - constexpr functions do not gurantee, constness or compiletime evalution of result. if called with args known at compile time, it is evaluted at compile time, else at run time.
    - constexpr objects and functions may be used in a wider range of contexts than non-constexpr objects and functions.- constexpr is part of an object’s or function’s interface.

### Make const member functions thread safe.
    - const member function using mutable data members is not thread safe.
    - using mutex or atomic as a class member makes class non-copiable and non-movable, use with care.
    - For a single variable or memory location requiring synchronization, use of a std::atomic is adequate, but once you get to two or more variables or memory locations that require manipulation as a unit, you should reach for a mutex. 

### Understand special member function generation.
    - compiler generates following functions if not declared by programmer and used.
        - default ctor (only if no parameterized ctor is defined.) :  `A()`
        - copy ctor : `A(const A&)`
        - copy = : `A& operator=(const A&)`
        - destructor : `~A()`
        - move ctor -- c++11 : `A(A&&)`
        - move =  -- C++11 : `A& operator=(A&&)` 
    - c++98 rule of three, c++11 - rule of five.
    - move ops are not generated if,
        - dotr or any of copy ops defined.
        - if one move op is there, other is not auto generated. so define both.
    - if move ops are declared, copy ops are not auto generated.    
    - use = default suffix to define default implementation same as compiler generated.

# Smart Pointers
    - raw pointers are hard to manage, leads to dangling pointers, memory leaks, seg faults, undefined behaviours if not handled with care. No proper ownership semantics.
    - smart pointers tries to solve that.

### Use std::unique_ptr for exclusive-ownership resource management.
    - exclusive ownership semanntics.
    - size almost same as raw pointers.
    - RAII - deletes the owned pointer when goes out of scope, even in case of exceptions.
    - no copy allowed, as copying makes unique ownership shared.
    - only moves are allowed.
    - by default use delete to delete raw pointers, can provide custom deleters.
    - if custom deleter provided, size of unique pointer increased by size of function pointer, or functor. lamda can also be used, which does no affect size.
    - best fit as return value of factory functions. can be converted to shared pointer implicitly.
    - unique_ptr<T[]> for arrays.
    - make_unique as factory function to create unique ptrs.
    - unique_ptr<T, void(f*) (T*)> up (new T(), &f); 

### Use std::shared_ptr for shared-ownership resource management.
    - shared ownership semantics, similar to garbage collection
    - no shared_ptr<T[]> array variant
    - size mostly double of unique pointer, due to control block with every shared pointers.
    - contains reference count in control block, if it goes to 0, deleter is called. 
    - default deleter is `delete`, custom deleter can be provided.
    - following operations change reference count
        - ctor : creates control block, with ref count 1.
        - copy ctor : rc++
        - copy = op : rc++ dest, rc-- source, if rc goes to 0, delete it.
        - move ctor : moves the pointer & control block from source. no change in rc. source set to null.
        - move = op : moves the pointer & control block from source
    - shared pointer rc incr/decr is atomic, so slow.
    - enable shared from this, when you want to create shared pointer from this, uses CRTP. call shared_from_this.
    - control block created when shared pointer is created with
        - make_shared
        - from unique pointer
        - raw pointers.
    - make shared does not provide option to give custom deleter. custom deleter is not part of type.

### Use std::weak_ptr for std::shared_ptr-like pointers that can dangle.
    - weak pointer, is like observer. created from shared pointer, does not take part in shared ownership & does not affect reference counting.. use same control block as shared pointer.
    - to use weak pointer, can check if it is expired. or call lock function which returns associated shared pointer.
    - no * or -> ops on weak pointer.
    - handles dangling pointers scenario well.
    - can help breaking cyclic dependency.
    - control block have weak count to specify weak references.

### Prefer std::make_unique and std::make_shared to direct use of new.
    - make functions - takes arbitory number of args, perfect forwards them and creats dynamically allocated objects and return smart pointers to it.
    - make_unique(c++14), make_shared & allocate_shared (uses custom allocater)
    - make calls are exception safe, when called as part of function args, make shared does single allocation for object and control block, thus saving extra allocation call.
    - make functions does not allow providing custom deleter.
    - does not work with { }, as {} cant be perfect forwarded.
    - make shared has 2 other issues.
        - when using with custom allocater, and class specific new & delete it does not work.
        - and as long as weak count does not goes to 0, control block is not deleted & thus when using make shared, object is also not deleted as long as there is any waek pointer reference.

### When using the Pimpl Idiom, define special member functions in the implementation file.
    - Pimpl idiom helps with compilation times & indirection.
    - incomplete type : declared but not defined.
    - using pimp idiom with unique pointer is go to way in modern C++, but it throws error if destructor is not defined in .cpp file. because, Impl struct is incomplete type & compiler defined dtor is public & inline. so in .h file, when it tried to delete using default deleter, static assert fails due to Impl struct being incomplete type.
    - declare dtor in header and use default implementation in .cpp file.
    - same applies for copy & move ops. 
    - shared pointers does not need this when used for pimpl, because for unique deleter is part of type & for shared it is not.

# Rvalue References, Move Semantics, and Perfect Forwarding
    - Move Semantics : dictates how types are moved.
    - perfact forwarding - enables variadic templates to pass arbitary number of function args as it is.


### std::move & std::forward
    - move is a function template that does the cast and convert arg to rvalue unconditionally.
        ```
        template<typename T>
        decltype(auto) move(T&& t)
        {
            return static_cast<remove_reference<T>::type &&>(t);
        }
        ```cpp
    - move request on const convert them silently to copy, so objects you want to move should not be const.
    - forwars is conditional cast - cast to rvalue if and only if args passed is rvalue itself.
    - move only needs an arg, forward need both template type and arg.

###  Distinguish universal references from rvalue references.
    - && can mean, rvalue reference or universal reference.
    - rvalue reference only binds to rvalue, universal reference can bind to lvalue & rvalues both(typically anything).
    - && is universal reference only in cases where there is type deduction, so T&&  or auto&&.
    - check reference collapsing

### Use std::move on rvalue references, std::forward on universal references.
    - use std::move/std::forward to rvalue/universal reference when being used last time.
    - apply it to those references when being used as return value.
    - Return Value Optimizations : compiler does copy elision when returning local object by value. using move or forward in this case is not advisable, it causes copy & stop compiler applying RVO.

### Avoid overloading on universal references
    - function taking universal references are greediest of all functions, almost all calls macth with this.
    - so do not overload with universal reference as args.


### Alternatives to overloading on universal references.
    - do not use overloads.
    - use const &
    - use pass by value
    - all above are not as efficient as universal reference.
    - use tag dispatch 
    - extra args to identify when to use specific overloads.
    - std::false_type & std::true_type compile time constants/types specifying true & flase.
    - tag dispatch solves issue with function overloads with single non overloaded function as a interface, but it does not solve problems with ctors taking universal references.
    - use std::enable_if, std::enable_if gives you a way to force compilers to behave as if a particular template didn’t exist. Such templates are said to be disabled.
    - need to use enable if with is_base_of, to handle both base & derived class ctors, std::is_same only handles base class

        ```
        template<
        typename T,
        typename = typename std::enable_if<
                    !std::is_base_of<Person,
                                    typename std::decay<T>::type
                                    >::value
                >::type
    >
    explicit Person(T&& n);
        ```

### reference collapsing
    - reference to reference are illegal.
    - compiler allows it in certain context, type deduction is one such case.
    - If either reference is an lvalue reference, the result is an lvalue reference. Otherwise (i.e., if both are rvalue references) the result is an rvalue reference.
        - Type deduction distinguishes lvalues from rvalues. Lvalues of type T are deduced to have type T&, while rvalues of type T yield T as their deduced type.
        - Reference collapsing occurs.
    - Reference collapsing occurs in four contexts: template instantiation, auto type generation, creation and use of typedefs and alias declarations, and decltype.

### Assume that move operations are not present, not cheap, and not used.
    - move ops may not be present. : old code bases/types
    - may be as efficient as copy only. : SSO for strings, std::array for example.
    - may be not declaered with noexcept, so not used in strong exception safety context.
    - but still many cases it will work fine.

### perfect forwarding failure cases.
    - works in most cases, but not with pointers or pass by values.
    - used for template functions taking universal references mostly.
    - fails when function forwarding & function begin forwarded to does diff things for same expr.  
    - cases :
        - Compilers are unable to deduce a type
        - Compilers deduces the wrong type.
    - braces initilizers
    -  0 or NULL
    - Declaration-only integral static const and constexpr data members
    - Overloaded function names and template names
    - Bitfields

# lambda
    - lambda is an expression.
    - closure is run time object that is created for lambda.
    - closure class is a class instantiated for lambda.
    - [](){} is a simplest lambda, return type is auto deduced, can be provided as trailing types if needed.
    - lambdas are const by default, so captures can not be modified.

### Avoid default capture modes.
    - lambda syntax , [](){} - simplest lambda
    - [=] default value capture  : susecptible to dangling refs
    - [&] default reference capture
    - Default by-value capture is susceptible to dangling pointers (especially this), and it misleadingly suggests that lambdas are self-contained.

### Use init capture to move objects into closures.
    - allows to move
    - allows to specify data member for closure & expr to initilize this data member.
    - [pw = move(apw)](){use pw} - pw here is member of closure class. from c++14, C++11 does not allow expr used in lambda capture
    - another name for init capture is generalized lambda capture.
    - by default operator() created for closure is const, so data member captured can not be modified in lambda.

### Use decltype on auto&& parameters to std::forward them.
    - generic lambdas, use of auto as a parameter.
        - auto f = [](auto x){ return process(x); };
        - operator () is a template for the closure class for generic lambdas,
        - generic lambda accepting universal ref 
            ```
            auto f = [](auto&& x)
                     { return process(std::forward<decltype(x)>(x)); }
            ```
        - generic variadic lambda
            auto f = [](auto&&... xs)
                     { return process(std::forward<decltype(xs)>(xs)...); };

### Prefer lambdas to std::bind.
    - Compared to lambdas, then, code using std::bind is less readable, less expressive, and possibly less efficient.

# The Concurrency API

### Prefer task-based programming to thread-based.
    - thread t([](){return val;}) - thread based - you need to  manage scheduling
    - auto fut = async([](){return val;}) - task based  - scheduler can manage it for you, no need to worry about oversubscription, load balancing etc.
    - threads can not return values, and if exception is thrown, std::terminate gets called, terminating program.
    - task based approach provides future to return value & can catch exception also.
    - std::async can run task in the same thread, if needed based on scheduling context.

### Specify std::launch::async if asynchronicity is essential.
    - 2 launch policy
        1. launch::async - runs task in seperate thread
        2. launch::deferred - runs only when get or wait function on future is called.
    - default policy is OR of both policy.
    - use policy launch::async if async is essential.


### Make std::threads unjoinable on all paths.
    - if the destructor for a joinable thread is invoked, execution of the program is terminated.
    - thread is unjoinable if
        - default constructed
        - moved
        - joined
        - detached.
    - priority/affinity etc can only be set using native api.
    - native_handle provides handle to native threads, i.e pthreads
    - use RAII type class to handle thread destruction
        2 options.
            1. join -  can lead to performance issue
            2. detach  - can lead to hard to debug UB.

### Be aware of varying thread handle destructor behavior.
    - future & thread both are handle to thread of execution, but dtor behaviour are diff.
    - destructor of joinable thread calls terminate.
     but dtor for future, calls detach/join, never terminates the program.
    - future and promise are 2 ends of comm channel, future with caller & promise with callee.
    - result can not be stored with caller or callee, callee stack can be unwinded, future can be moved, transferred/shared. it is kept in shared state.
    - The shared state is typically represented by a heap-based object, but its type, interface, and implementation are not specified by the Standard.
    - dtor behaviour for futures
        - dtor of last future referring to non-deferred task running with async policy does implicit join
        - other futures are just deleted, similar to detach
    - “Futures from std::async block in their destructors.”

     ![Thread channel](image.png)

### Consider void futures for one-shot event communication.
    - using condition variable & mutex to notify other task has 2 problems
        - If the detecting task notifies the condvar before the reacting task waits, the reacting task will hang. 
        - The wait statement fails to account for spurious wakeups 
        - also reacting task/consumer may do not have a way to check satisfying condition,
    - another way to is shared boolean flag, but it leads to blocking/busy wait.
    - can use flag and mutex both to avoid busy waiting.
    - use a promise<void> & future<void> to handle one shot comminication.
    - can be extended to multiple threads using shared_future.
    - auto sf = p.get_future().share() - 
    - limitations
        - only for single one shot communications.
        - needs shared state between promise & future so heap allocation.

### Use std::atomic for concurrency, volatile for special memory.
    - Once a std::atomic object has been constructed, operations on it behave as if they were inside a mutex-protected critical section, but the operations are generally implemented using special machine instructions that are more efficient than would be the case if a mutex were employed.
    - volatile does not have same properties as atomic. what volatile specifies is that volatile memory does not behave normally.
    -  redundant loads and dead stores
    - most common kind of special memory is memory used for memory-mapped I/O. sensors, peripherals etc.
    - volatile is the way we tell compilers that we’re dealing with special memory. Its meaning to compilers is “Don’t perform any optimizations on operations on this memory.”

# MISC

### Consider pass by value for copyable parameters that are cheap to move and always copied.
    - For copyable, cheap-to-move parameters that are always copied, pass by value may be nearly as efficient as pass by reference, it’s easier to implement, and it can generate less object code.
    - For lvalue arguments, pass by value (i.e., copy construction) followed by move assignment may be significantly more expensive than pass by reference followed by copy assignment.
    - Pass by value is subject to the slicing problem, so it’s typically inappropriate for base class parameter types.


### Consider emplacement instead of insertion.
    - emplace_back instead of push_back,
    - emplace_front, emplace.
    - saves on extra copy ctors.
    - Insertion functions take objects to be inserted, while emplacement functions take constructor arguments for objects to be inserted. This difference permits emplacement functions to avoid the creation and destruction of temporary objects that insertion functions can necessitate.
    - If all the following are true, emplacement will almost certainly outperform insertion:
        - The value being added is constructed into the container, not assigned. 
        - The argument type(s) being passed differ from the type held by the container. 
        - The container is unlikely to reject the new value as a duplicate
    - Emplacement functions use direct initialization, which means they may use explicit constructors


# concurrency:

### Synchronization:
    - sometimes you don’t just need to protect the data, you also need to synchronize actions on separate threads.

### How to wait?
    - check the shared flag, is set or not, resource wastage, waiting thread blocks mutex and waste resources.
    - sleep for some time and check again: std::this_thread::sleep_for(std::chrono::milliseconds(100)) - it is hard to get sleep time right,
    - conditional varialble - 
        - std::conditional_variable - needs a mutex to work with
        - std::conditional_variable_any - works with anything mutex like.

    - cond.wait(ulock, [](){return is_data_ready();}) -- consumer thread, waiting for data, needs to be used with unique lock, can not use lock guard, as when thread goes to wait, it unlocks the mutex, lock-guard does not have the functionality.
    - cond.notify_one()/ cond.notify_all() --  
    - wait - unlocks the mutex and goes on wait, when notified, locks the mutex, check condition if true process, if false unlocks again and goes to wait.
    - spuriopus wakes - thread wakes without notify, needs condition to handle this,
    - std::condition_variable::wait is an optimization over a busy-wait.

    - Waiting for one-off events with futures
    - unique futures (std::future<>) and shared futures (std::shared_future<>)
    - You use std::async to start an asynchronous task for which you don’t need the result right away. Rather than giving you a std::thread object to wait on, std::async returns a std::future object, which will eventually hold the return value of the function. When you need the value, you just call get() on the future, and the thread blocks until the future is ready and then returns the value
    - auto fut = async(func, args);
    - use launch policy deferred  - call only when get or wait on future gets called, or async - new thread
    - auto fut = async(func, args)
    - auto fut = packaged_task(func, args).get_future();
    - auto fur = promise<T>().get_future() - 
    - if there is an exception thrown for the function which future is waiting for, get rethrows an exception,
    - promise.set_value(), promise.set_exception()
    - shared_future - waiting from multiple threads
    - future - movable, shared_future - movable & copyable.
    - Instances of std::shared_future that reference some asynchronous state are constructed from instances of std::future that reference that state.

    - Waiting with a time limit
    - do not wait indefinately,
    - timeout
            - duration based - 30 second -- wait_for
            - absolute - till 5 PM -- wait_until