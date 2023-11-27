- Logical vs bitwise constness.
- mutable to allow member to be modified in const member function.
- implement non-const [] operator as a calling const [] operator with casting result to non-const & *this to const to avoid code duplication.
- data members of objects are initialized before the body of constructor is entered.
- initialize member in initizalizer list, when done inside ctor, it is default initialisation + assignment.
- order of initialization: base classes, member variables in order of declaration.
- relative order of initialization of non local static objects defined in different translation unit is undefined.

### Default functions
- compiler generates default ctor, dtor, copy ctor & copy =, even if not defined, if needed.
- if those functions are not valid, you want them to be not called ever, declare them private & do not define it. - in CPP11 - use = delete & make them public.
- can use uncopyable base class to make it compile time error.
- factory function: a function that returnes base class pointer to newly created derived class object.
- virtual destructor
- when derived class object is deleted with a base class pointer with non virtual destructor, results are undefined.
- if derived class object deleted with base class pointer with non virtual dtor, derived class dtor never called & derived class object is not deleted.
- any class with virtual function should have a virtual dtor.
- when there is no virtual function & defining dtor virtual adds to the size of objects unnecessarily.
- vptr & vtable, vptr point to vtable.  Vtable list of function pointers.
- pure virtual function makes class abstract, can't instantiate it.
- pure virtual dtor virtual ~ cls() =0
- must provide defn for pure virtual dtor.
- prevent exception from leaving dtor
- throwing dtor may lead to more than 1 active exception, that is not valid in cpp runtime, it is undefined behaviour or termination.
- 2 ways to handle
  - abort
  - swallow
- Never call virtual function from dtor or ctor
- during base class ctor, virutal function never go down to derived class
- assignment operator
  -  Right associative
  -  return *this
  -  handle self assignment & exception safety
  -  best way to handle is copy swap idiom
- copy all parts including base class
- resources management
- RAII - resources aquisition is initialization
- use objects to manage resources.
- copying behaviour for raai resources.
- options:
       - prohibit copy - deleted copy ctors & =
       - reference count the resource - shared ptr
       - deep copy - RAAI 
       - transfer ownership of the underlying resources - unique_ptr / auto ptr
- provide access to underlying resources pointer
- get function in shared ptr , * & -> operator
- new & delete
    - when new called, operator new allocate memory, and then call ctors
    - delete calls destructor and deallocates the memory
    - if you use [] in new must use [] in delete , else undefined behaviour catches you
    - don't use dynamically allocated array, prefer vector & string

- make ctor explicit to prohibit implicit conversion of your class object
- store newed objects in smart pointer in standalone statement

## design

### Interface: easy to use correctly & hard to use incorrectly.
- when in doubt do as the ints do
- consistency in design and interface
- Treat class design as type design
- how objects of new type created & destroyed
- how initialisation differs from assignment
- what it means to pass by value
- allowed values
- inheritance graph for type
- conversion, implicit & explicit
- function & operator for class/type
- what standard funcs to be disallowed
- access
- undeclared interface, gurrantes etc

### Pass by reference to const vs pass by values
- pass by value calls copy ctor
- pass by ref saves copy ctor calls
- slicing problem

- Never return pointer or reference to local stack object from function

### Data memeber must be private
- fine grained access control
- encapsulation
- class invariant is always maintained

- Prefer non member non friend function to member function
- Declare non member function when type conversation should apply to all params
- Provide non throwing swap implementation
- You can totally specialize std template function for used defined types, can't add anything to std

- postpone var defination as long as possible
- minimize casting
   - C type Casts
        T(exp)
        (T) exp
   - C++ cast
   - const_cast : cast away constness
   - dynamic_cast : check if object belongs to inheritance heirarchy, used for safe downcasting, runtime expensive
   - reinterpret_cast : low level implementation dependent casting
   - static_cast : normal type conversion, implicit & forced

- avoid dynamic_cast, it is very slow

- Avoid returning handles to internal of objects
- data members are only as encapsulated as most accessible function returning a reference to it.
- returning internal can lead to encapsulation prob, incorrect constness or dangling pointer/reference so avoid it.
- writing exception safe code:
    - no resources leak
    - no data corruption
- Types of exception safety
    - basic: object is in valid state even if exception thrown
    - strong: object is in valid starting state before function called if exception thrown
    - nothrow: function will not throw
- Copy & swap idiom: to avoid resources leaks & achieving exception safety
- Use pimpl idiom & smart pointers.
- If functions have side effects hard to guarantee strong gurrantee.

### Inline function:
- Request to compiler
- function body inlined instead of function call,
- good for small function
- function defined in class are inline

### OOP 
- public inheritance is "is a" relationship.
- everything true for base class need to be true for derived class.

#### Name hiding
- scope of derived class is nested inside its base class
- name in derived class hides name in base class, does not matter if the type is same, or virtuality or overloaded.
- using base:f to unhide the functions.
- inline forwarding can also work.

#### Inheritance of interface Vs implementation.
- pure virtual function : makes class abstract, derived class must implement, derived class inherit only interface

- virtual function : derived class can override, gets interface & implementation
- non-virtual function : defines behaviour invariant, behaviour should not change in derived class.
- overloaded function


#### Template method pattern
- non virtual interface idiom: make public interface public non virutal, non virtual function calls virutal private functions
- derived class defines how function is called, base class defines when it will be called.

#### Strategy pattern;
- instead of using virtual function to change behaviour, use strategy function using function pointers/std:: function as member,
- less encapsulation, more flexibility.
- std:: function allows stategy pattern to use any function like callable object , member function, non member function, functor, lambda etc


- never redefine inherited non virtual member function in derived.
- non virtual function are statically bound, depends only on variable type
- virtual function are dynamically bound, depends on object type var pointing to.

- never redefine function's inherited default parameter value
- virtual function are dynamically bound, but default params are statically bound .
- multiple inheritance: diamond problem,
- use virtual inheritance, not free, use to break diamond prob
- private inheritance for is implemented in terms of, implementation, use composition instead when possible


## Templates
- explicit vs implicit interface
- compile time vs runtime polymorphism
Template provide implicit interface and compile time polymorphism
Turing complete, stl is best example

- Member functions of class templates are implicitly instantiated only when used, so you should get the full 300 member functions only if each is actually used.
- base template class function not available in derived template class. By default. Bring it in scope.
- template dependent typename. When type is dependent on template parameter use typename to define it.
- complete specialization.
- template instantiation
- partial specialization
- non type template parameters like size
- member function template
- generalized copy ctors, for implicit conversion from one type to other type like smart pointers work
- Use member function templates to generate functions that accept all compatible types.

-  If you declare member templates for generalized copy construction or generalized assignment, you'll still need to declare the normal copy constructor and copy assignment operator, too.
-  implicit type conversion functions are never considered during template argument deduction.

### iterators
-  Input iterators can move only forward, can move only one step at a time, can only read what they point to, and can read what they're pointing to only once. They're modeled on the read pointer into an input file; the C++ library's istream_iterators are representative of this category.
-  Output iterators are analogous, but for output: they move only forward, move only one step at a time, can only write what they point to, and can write it only once. They're modeled on the write pointer into an output file; ostream_iterators epitomize this category.
-  forward iterators. Such iterators can do everything input and output iterators can do, plus they can read or write what they point to more than once. fwd list has it l.
-  Bidirectional iterators add to forward iterators the ability to move backward as well as forward. Iterators for the STL's list are in this category, as are iterators for set, multiset, map, and multimap.
-  random access iterators. These kinds of iterators add to bidirectional iterators the ability to perform “iterator arithmetic,” i.e., to jump forward or backward an arbitrary distance in constant time.
-  traits allow you to get information about a type during compilation.
-  template metaprogram is a program written in C++ that executes inside the C++ compiler.
-  generative programming.
-  Turing complete.


### New & delete
- handles memory allocation & de allocation.
- new/delete & new []/delete [] are diff set of operators later one used for array allocation.
- new handler function is called when operator new can't allocate asked memory, you can set it. Using set new handler function.
- write custom new & delete to
    -  improve efficiency space /performance
    -  detection of usage errors
    -  logging/tracing usage like valgrind
    -  alignment is something to look into when writing your own new & delete
- Implementing a conformant operator new requires having the right return value, calling the new-handling function when insufficient memory is available and being prepared to cope with requests for no memory. You'll also want to avoid inadvertently hiding the “normal” form of new, though that's more a class interface issue than an implementation requirement.
- operator new needs to return valid pointers even for 0 byte memory request
- it needs to return valid value or throw exception bad_alloc
- base class new is inherited
- make sure deleting nullptr is valid
- handle class specific request & other pass it to global operators
- placement new , new taking extra param, mostly it is void * to specify memory where to allocat objects, std provides it, vector does use it.
- placement new needs to have matching placement delete or memory leak when exception is thrown during ctor .
- STL do type checking at compile time,
- iterators decouple algorithms from iterators.
- priority queue do not have begin and end iterators.
- set and multiset iterators are const


- move: i no longer need this value!!

