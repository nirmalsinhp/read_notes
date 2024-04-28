CPP

OOP 
- public inheritance is "is a" relationship
- composition for reuse
- Non virtual interface
- if base class have a virtual function, dtor needs to be virtual.
- if polymorphic base class pointer pointing to derived class, and delete on pointer is called, dotr of derived class is never called and behaviour is undefined.
- if dtor throws an exception, program terminates.
- adding pure virtual function makes class abstract, can not create object of abstract class.
    virtual void f() = 0;
- pure virtual dtor can have a body.
- derived class must implement pure virtual function else that class is also abstract
- make public functions non virtual, make virtual functions private
- do not call virtual functions in ctor/dtor
- member variables are initialized before ctor body starts, do it in initializaer list.
- default value is static time construct, do not use default value for overriden functions.
- make non - leaf classes abstract.
- dynamic_cast is slow, and may be pointing to underlying design issue.
- name in derived class, hides other symbols with same name in base class. you can use using to get them in same scope.
- if overriding a virtual function, write override to show the intent, saves from subtle errors & gotchas.
- give each class single cohesive responsibility
- virtual function is dynamic dispatch, uses vptr & vtable. every class have a vtable & vpointer pointing to it in every object.
- vptr increase the size by pointer size.
- empty base optimization.
- object slicing, sizeof gives sizeof static type.
- diamond problem & virtual inheritance.


RAAI 
- Respource Aquisition in initialization.
- aquire resource in ctor & delete them in dtor.
- order of construction : base class, member variable, derived class
- order of destruction : reverse of ctor.
- special functions : 


Exception safe
- code which is robust.
    - no resource leak
    - no data corrpution
    - no invalid state.
- gurantees
- basic : object is in valid state after exception is thrown.
- strong : object is in same state as before method started which throws exception
- noexceot : no exception will be thrown.

Smart pointers

ADL

Function overload resolution

Templates

Type Deduction

Design patterns

STL
- check
    - STL iterators & invalidation
    - map::merge, map::extract, list::splice, list::merge, vector::front, vector::back
    - STL associate container - find, count, equal_range
    - parralle execusion, transform_reduce
    - std::span

iterators

Algos

MultiThreading

Lambda

Idioms
- Non virtual interface idiom
- template method patter
- Pimpl Idiom
- copy and swap
- virtual ctor idiom
- erase-remove idiom
- Rule of 0/3/5
- CRTP
- SFINAE



Checks
- init if 
- structured binding
- string_view
- nested namespaces
- class template argument deduction
- visit
- constexpr
- fold expressions
- span
- optional
- any
- constexpr if
- virtual constructors
- covariant types
- using reinterpret_cast to access private member of another class object(bad trick, might be handy)

- do not make performance claims without measurements & benchmarks.
- glibs and nolibc (small and fast)
- syscall

- NVRO can not be applied to sub object



-- ask clarifying questions, solve the right problem
-- do it with no or minimum hints
-- complete all, check your code, check for edge cases.
-- space and time complexity
-- time limit is 35 minutes, pace yourself
-- check code before submitting, run throguh edge cases.