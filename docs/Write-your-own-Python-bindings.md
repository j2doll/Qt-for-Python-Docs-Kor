
## 나만의 파이썬 바인딩 작성하기 (Write your own Python bindings)

- 원문:   [http://blog.qt.io/blog/2018/05/31/write-python-bindings/](http://blog.qt.io/blog/2018/05/31/write-python-bindings/)

- 번역:  [https://j2doll.github.io/Qt-for-Python-Docs-Kor/](https://j2doll.github.io/Qt-for-Python-Docs-Kor/)

안녕하십니까.

이전 블로그 포스트에서 우리는 Qt 라이브러리를 위한 파이썬 바인딩(Python Binding) 생성에 관한 주제를 다루었습니다.

하지만 오늘, 우리는 어떻게 당신이 프로젝트를 위한 바인딩을 만들 수 있는지 살짝 엿볼 것입니다.

Qt for Python에는 바인딩 생성 도구인 쉬보켄(Shiboken)도 포함되어 있습니다.

아래의 자료를 읽으면 간단한 C++ 라이브러리를 위한 Python 바인딩을 생성하는 방법을 이해할 수 있습니다. 다행히도 자신만의 맞춤 라이브러리를 사용하여 동일한 작업을 수행하는 것이 좋습니다.

여타 Qt 프로젝트와 마찬가지로 우리는 Shiboken에 대한 기여를 검토하게 되어 기쁘게 생각하며, 이를 통해 모두에게 개선될 수 있기를 바랍니다.

## 예제 라이브러리

![](https://qt-blog-uploads.s3.amazonaws.com/wp-content/uploads/2018/05/icecream.png)

For the purposes of this post, we will use a slightly nonsensical custom library called Universe. It provides two classes: Icecream and Truck.

Icecreams are characterized by a flavor. And Truck serves as a vehicle of Icecream distribution for kids in a neighborhood. Pretty simple.

We would like to use those classes inside Python though. A use case would be adding additional ice cream flavors or checking whether ice cream distribution was successful.

In simple words, we want to provide Python bindings for Icecream and Truck, so that we can use them in a Python script of our own.

We will be omitting some content for brevity, but you can check the full source code inside the repository under pyside-setup/examples/samplebinding.

## C++ 라이브러리

First, let’s take a look at the Icecream header:

```cpp
class Icecream
{
public:
    Icecream(const std::string &flavor);
    virtual Icecream *clone();
    virtual ~Icecream();
    virtual const std::string getFlavor();
 
private:
    std::string m_flavor;
};
```

and the Truck header:

```cpp
class Truck {
public:
    Truck(bool leaveOnDestruction = false);
    Truck(const Truck &other);
    Truck& operator=(const Truck &other);
    ~Truck();
 
    void addIcecreamFlavor(Icecream *icecream);
    void printAvailableFlavors() const;
 
    bool deliver() const;
    void arrive() const;
    void leave() const;
 
    void setLeaveOnDestruction(bool value);
    void setArrivalMessage(const std::string &message);
 
private:
    void clearFlavors();
 
    bool m_leaveOnDestruction = false;
    std::string m_arrivalMessage = "A new icecream truck has arrived!\n";
    std::vector m_flavors;
};
```

Most of the API should be easy enough to understand, but we’ll summarize the important bits:

- Icecream is a polymorphic type and is intended to be overridden
- getFlavor() will return the flavor depending on the actual derived type
- Truck is a value type that contains owned pointers, hence the copy constructor and co.
- Truck stores a vector of owned Icecream objects which can be added via addIcecreamFlavor()
- The Truck’s arrival message can be customized using setArrivalMessage()
- deliver() will tell us if the ice cream delivery was successful or not

## Shiboken typesystem

To inform shiboken of the APIs we want bindings for, we provide a header file that includes the types we are interested in:

```cpp
#ifndef BINDINGS_H
#define BINDINGS_H
#include "icecream.h"
#include "truck.h"
#endif // BINDINGS_H
```

In addition, shiboken also requires an XML typesystem file that defines the relationship between C++ and Python types:

```xml
<?xml version="1.0"?>
<typesystem package="Universe">
    <primitive-type name="bool"/>
    <primitive-type name="std::string"/>
    <object-type name="Icecream">
        <modify-function signature="clone()">
            <modify-argument index="0">
                <define-ownership owner="c++"/>
            </modify-argument>
        </modify-function>
    </object-type>
    <value-type name="Truck">
        <modify-function signature="addIcecreamFlavor(Icecream*)">
            <modify-argument index="1">
                <define-ownership owner="c++"/>
            </modify-argument>
        </modify-function>
    </value-type>
</typesystem>
```

The first important thing to notice is that we declare "bool" and "std::string" as primitive types.


A few of the C++ methods use these as parameter / return types and thus shiboken needs to know about them. It can then generate relevant conversion code between C++ and Python.
Most C++ primitive types are handled by shiboken without requiring additional code.

Next, we declare the two aforementioned classes. One of them as an “object-type” and the other as a “value-type”.

The main difference is that object-types are passed around in generated code as pointers, whereas value-types are copied (value semantics).

By specifying the names of the classes in the typesystem file, shiboken will automatically try to generate bindings for all methods declared in the classes, so there is no need
to mention all the method names manually…

Unless you want to somehow modify the function. Which leads us to the next topic: ownership rules.

Shiboken can’t magically know who is responsible for freeing C++ objects allocated in Python code. It can guess, but it’s not always the correct guess.
There can be many cases: Python should release the C++ memory when the ref count of the Python object becomes zero. Or Python should never delete the C++ object assuming that it will
be deleted at some point inside the C++ library. Or maybe it’s parented to another object (like QWidgets).

In our case the clone() method is only called inside the C++ library, and we assume that the C++ code will take care of releasing the cloned object.

As for addIcecreamFlavor(), we know that a Truck owns an Icecream object, and will remove it once the Truck is destroyed. Thus again, the ownership is set to “c++.”
If we didn’t specify the ownership rules, in this case, the C++ objects would be deleted when the corresponding Python names go out of scope.

## Building

To build the Universe custom library and then generate bindings for it, we provide a well-documented, mostly generic CMakeLists.txt file, which you can reuse for your own libraries.

It mostly boils down to calling “cmake .” to configure the project and then building with the tool chain of your choice (we recommend the ‘(N)Makefiles’ generator though).

As a result of building the project, you end up with two shared libraries: libuniverse.(so/dylib/dll) and Universe.(so/pyd).
The former is the custom C++ library, and the latter is the Python module that can be imported from a Python script.

Of course there are also intermediate files created by shiboken (the .h / .cpp files generated for creating the Python bindings). Don’t worry about them unless you need to
debug why something fails to compile or doesn’t behave as it should. You can submit us a bug report then!

More detailed build instructions and things to take care of (especially on Windows) can be found in the example README.md file.

And finally, we get to the Python part.

## Using the Python module

The following small script will use our Universe module, derive from Icecream, implement virtual methods, instantiate objects, and much more:

```python
from Universe import Icecream, Truck
 
class VanillaChocolateIcecream(Icecream):
    def __init__(self, flavor=""):
        super(VanillaChocolateIcecream, self).__init__(flavor)
 
    def clone(self):
        return VanillaChocolateIcecream(self.getFlavor())
 
    def getFlavor(self):
        return "vanilla sprinked with chocolate"
 
class VanillaChocolateCherryIcecream(VanillaChocolateIcecream):
    def __init__(self, flavor=""):
        super(VanillaChocolateIcecream, self).__init__(flavor)
 
    def clone(self):
        return VanillaChocolateCherryIcecream(self.getFlavor())
 
    def getFlavor(self):
        base_flavor = super(VanillaChocolateCherryIcecream, self).getFlavor()
        return base_flavor + " and a cherry"
 
if __name__ == '__main__':
    leave_on_destruction = True
    truck = Truck(leave_on_destruction)
 
    flavors = ["vanilla", "chocolate", "strawberry"]
    for f in flavors:
        icecream = Icecream(f)
        truck.addIcecreamFlavor(icecream)
 
    truck.addIcecreamFlavor(VanillaChocolateIcecream())
    truck.addIcecreamFlavor(VanillaChocolateCherryIcecream())
 
    truck.arrive()
    truck.printAvailableFlavors()
    result = truck.deliver()
 
    if result:
        print("All the kids got some icecream!")
    else:
        print("Aww, someone didn't get the flavor they wanted...")
 
    if not result:
        special_truck = Truck(truck)
        del truck
 
        print("")
        special_truck.setArrivalMessage("A new SPECIAL icecream truck has arrived!\n")
        special_truck.arrive()
        special_truck.addIcecreamFlavor(Icecream("SPECIAL *magical* icecream"))
        special_truck.printAvailableFlavors()
        special_truck.deliver()
        print("Now everyone got the flavor they wanted!")
        special_truck.leave()
```

After importing the classes from our module, we create two derived Icecream types which have customized “flavours”.

We then create a truck, add some regular flavored Icecreams to it, and the two special ones.

We try to deliver the ice cream.
If the delivery fails, we create a new truck with the old one’s flavors copied over, and a new *magical* flavor that will surely satisfy all customers.

The script above succinctly shows usage of deriving from C++ types, overriding virtual methods, creating and destroying objects, etc.

As mentioned above, the full source and additional build instructions can be found in the project repository under pyside-setup/examples/samplebinding.

We hope that this small introduction showed you the power of Shiboken, how we leverage it to create Qt for Python, and how you could too!

Happy binding!


## 글

[글 목록](README.md)

