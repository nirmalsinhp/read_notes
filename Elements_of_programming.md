# Concepts
- abstract entity
- concrete entity
- attribute : property, measurement, quality of concrete entity
- identity
- value type
- datum 
- interpretation vs representation
- A value type is a correspondence between a species (abstract or concrete) and a set of datums
-  A datum corresponding to a particular entity is called a representation of the entity; the entity is called the interpretation of the datum. We refer to a datum together with its interpretation as a value.
- If a value type is uniquely represented, we implement equality by testing that both sequences of 0s and 1s are the same. Otherwise we must implement equality in such a way that preserves its consistency with the interpretations of its arguments. 
- function defined on a value type is regular if and only if it respects equality: Substituting an equal value for an argument gives an equal result. Most numeric functions are regular.
## Object
- An object is a representation of a concrete entity as a value in memory.An object has a state that is a value of some value type. The state of an object is changeable
- An identity token is a unique value expressing the identity of an object
## Procedures
- A procedure is a sequence of instructions that modifies the state of some objects; it may also construct or destroy objects.
- inpput/output : argument and return values
- local state : automatic variables
- global state : global variables
- own state : function static variables.
- A computational basis for a type is a finite set of procedures that enable the construction of any other procedure on the type
- type is regular if and only if its basis includes equality, assignment, destructor, default constructor, copy constructor, total ordering,[2] and underlying type.[
## Regular procedures
- A procedure is regular if and only if replacing its inputs with equal objects results in equal output object.
- A functional procedure is a regular procedure defined on regular types, with one or more direct inputs and a single output that is returned as the result of the procedure.
- A homogeneous functional procedure is one whose input objects are all the same type.
- Domain : types of inputs
- CoDomain : type of outputs.
## Concepts
- A procedure using a type depends on syntactic, semantic, and complexity properties of the computational basis of the type.
- The utility of a software component, such as a library procedure or data structure, is increased by designing it not in terms of concrete types but in terms of requirements on types expressed as syntactic and semantic properties. We call a collection of requirements a concept. Types represent species; concepts represent genera.
- A type attribute is a mapping from a type to a value describing some characteristic of the type
- A type constructor is a mechanism for creating a new type from one or more existing types. 
- A type function is a mapping from a type to an affiliated type.
- Somewhat more formally, a concept is a description of requirements on one or more types stated in terms of the existence and properties of procedures, type attributes, and type functions defined on the types
- An abstract procedure is parameterized by types and constant values, with requirements on these parameters.

# Transformations and Their Orbits
 - Define transformation as a unary regular function from a type to itself. Successive applications of a transformation starting from an initial value determine an orbit of this value. Depending only on the regularity of the transformation and the finiteness of the orbit, we implement an algorithm for determining orbit structures that can be used in different domains.
## Transformation
- homogeneous predicates : *T × ... × T → bool*
- Operations :  *T × ... × T → T*
- A predicate is a functional procedure returning a truth value.A homogeneous predicate is one that is also a homogeneous function
- An operation is a homogeneous function whose codomain is equal to its domain.
- Unary Operations which are transformations: y = f(x)
- composable : f(f(f(x)))
  ```
  template <typename F, typename N>
    requires(Transformation(F) && Integer(N))
  Domain(F) tranformation(Domain(F) x, N n, F f)
  {
    while(n != N(0))
    {
        n = n - 1;
        x = f(x);
    }
    return x;
  }
   ``` 

## Orbits
- orbits: elements reachable from a starting element by repeated applications of the transformation.
- reachable y =  f^n^(x)
- cyclic x = f^n^(x)
- If y is reachable from x under f, the distance from x to y is the least number of transformation steps from x to y.
- Orbits have diff shapes
  - infinite -  no cycle or terminal element
  - circular - if x is cyclic
  - terminating - if it has terminal element 
  - P shaped - is not cyclic but has a cycle inside orbit.

## Collision Point