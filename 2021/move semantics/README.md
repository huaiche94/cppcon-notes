# Back to Basics: Move Semantics

https://www.youtube.com/watch?v=Bt3zcJZIalk&list=PLHTh1InhhwT4TJaHBVWzvBOYhp27UO7mI&ab_channel=CppCon

* Nicolai Josuttis

## 1. copy semantics (with C++98/C++03)

```
vector<string> coll;
coll.reserve(3); //reserve 3 mem in heap

string s{getData()}; #1
coll.push_back(s); #2
coll.push_back(getData()); #3 
```
#1 return data\0

#2 container "vector" has copy semantics. Whenever we pass an object into the conatiner, it creates a copy of the object. Here the vector creates a copy of "data"

#3 getData() would create a temporary copy, but at the end of this statement the temporary object will be destroyed after copying data to heap. But "memory allocation on heap is very expensive" -> improve with move semantics
#
## 2. move semantics (with C++11 compiler)
the cmopiler does not create a copy of the temporary string, just copy the address (shallow copy)
```
vector<string> coll;
coll.reserve(3); //reserve 3 mem in heap

string s{getData()};
coll.push_back(s);
coll.push_back(getData()); #1 faster!!

```
#1. Say the getData() returned "data2\0". The compiler does not create a copy on heap. This is a shallow copy. At the end of this statement, the destructor is called, the temporary variable on the stack which points to the "data2\0" will be destroyed. But it will not free any memory, because "data2\0" is moved.

in this case we should avoid object with names.

#
## 3. Object with Names
oftentimes you can't avoid objects with names


```
std::move() means "I no longer need this value here". It is just a mark.
```
#
## 4. Valid but Unspecified State

### 4.1 Re-using Objects after std::move()
```
1. read line-by-line using getline
2. swap
```
#
## 5.  Vectors With Move Semantics(C++ 11)
```
& lvalue reference
&& rvalue reference, no const because we steal the memory.
```
#
## 6. Strings with Move Semantics
steal memory from others
```
std::move(y) is equivalent to static_cast<string&&>(y)
```
#
## 7. Overloading on references
1.  void foo(const type&)
    * pass value without creating a copy
    * can bind to everything
2. void foo(type&)
    * pass named entity to return a value
    * only non-const named object(lvalues)
3. void foo(type&&)
    * pass value that is no longer needed
    * only objects without name or with move() (rvalues)
4. void foo(const type&&)
    * possible, but **semantic contradiction**
    * ususlly covered by **const** type &

### 1. don't use const if you later move
### 2. don't use const when returning by value (disabled move())


## QA:
1. why `string e1 = x;` calls copy assignment? not assignment operator?
* the `=` is only the assignment operator if the object e1 exists already. Here it is NOT an assignment. This is a way of initialization, which calls the constructor. THIS IS NOT AN ASSIGNMENT NEVER.

2. we can disable copy and allows move only type. [disable copying, enable moving]


<br>
<br>
<br>
<br>

# Move Semantics for Classes

## 1. examples
### impact on inheritance
if declare nothing, move constructor is automatically generated. If declare only the copy constructor but move constructor, move() will be falled back to copy()


## 2. disable move
declared virtual destructors disable move semantics
* declared virtual destructors disable move semantics
* dont declared destructors in derived classes
## 3. perfect forwarding (something to improve)
```
class C {
    void foo(const C&); //read only (binds to all vals)
    void foo(C&);       //write access (binds to non-const lval)
    void foo(C&&);      //move access (binds to non-const rval)
}

C v;
foo(v);  //calls foo(C&)

const C c;
foo(c);  //calls foo(const C&)


foo(C{});    //temp object, calls foo(C&&)
foo(std::move(v));   //move mark, calls foo(C&&)
```

if we are calling `foo` indirectly by `callFoo`, and we want the behavior keeps the same. It would be a nightmare if we have more than one parameters ( has to implement every combination )
```
void callFoo(C&& x) {
    foo(std::move(x)); //use move() to forward move semantics
}
```
#
## Perfect forwarding of parameters:
1. Template parameter
2. Declaring the parameter as && of the template parameter
3. std::forward<>()

terminology and definition of universal/forwarding reference(not clear) 50:00



## 4. universal reference
`emplace_back` applies perfect forwarding

`auto&&` is a universal/forwarding reference without being a template parameter

C++20: const views might not support iterating
