
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

이 게시물의 목적을 위해 Universe 라는 약간 무의미한 커스텀 라이브러리(customed library)를 사용하겠습니다. 그리고 두 종류의 클래스(class)를 추가하겠습니다: '아이스크림(Icecream)'과 '트럭(Truck)'.

아이스크림은 풍미(flavor)가 특징입니다. 그리고 트럭은 이웃 아이들을 위한 아이스크림 판매 차량 역할을 합니다. 아주 간단합니다.

파이썬 안에서 이 클래스들을 사용하고 싶습니다. 유스 케이스(use case)는 추가된 아이스크림 풍미를 추가하거나 아이스크림 배달(deliver)이 성공했는지 여부를 확인하는 것입니다.

간략히 얘기해서, 아이스크림과 트럭에 파이썬 바인딩(python binding)을 제공하여 나만의 파이썬 스크립트에서 사용할 수 있게 하려고 합니다.

간결성을 위해 일부 내용을 생략 하겠지만, pyside-setup/examples/samplebinding 에서 저장소 내의 전체 소스 코드를 확인할 수 있습니다.

## C++ 라이브러리

먼저 Icecream 헤더를 살펴 보겠습니다.

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

다음은 Truck 헤더입니다:

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

대부분의 API는 이해하기 쉬워야하지만 중요한 점들을 요약해 보겠습니다.

- Icecream은 폴리몰픽 타입(polymorphic type)이며 오버라이드(overridde) 할 수 있습니다.
- getFlavor()는 실제 파생된 타입(derived type)에 따라 flavor를 반환합니다.
- 트럭은 자신의 포인터(pointer)를 포함하는 값-타입(value type)입니다, 따라서 복사 생성자(copy constructor) 및 공동.
- 트럭은 addIcecreamFlavor()를 통해 추가할 수 있는 자신의 Icecream 객체의 벡터(vector)를 저장합니다.
- 트럭의 도착 메시지(arrival message)는 setArrivalMessage()를 사용하여 커스텀한 정의를 할 수 있습니다.
- deliver()는 아이스크림 전달이 성공했는지 여부를 알려줍니다.

## 쉬보켄 타입시스템 (Shiboken typesystem)

바인딩을 원하는 API를 shiboken에 알리기 위해 우리는 관심있는 타입(type)을 포함하는 헤더 파일을 제공합니다:

```cpp
#ifndef BINDINGS_H
#define BINDINGS_H
#include "icecream.h"
#include "truck.h"
#endif // BINDINGS_H
```

또한 shiboken은 C++과 Python 타입 간의 관계를 정의하는 XML 타입시스템 파일을 필요로 합니다.

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

첫 번째로 주의해야 할 점은 "bool" 및 "std::string"을 기본 타입(primitive type)으로 선언한다는 것입니다.

몇 가지 C++ 메소드(method)는 이것을 매개변수(parameter)/반환타입(return type)으로 사용하므로 shiboken은 이들에 대해 알아야 합니다. 그런 다음 C++과 Python 간에 적절한 변환 코드를 생성할 수 있습니다.

대부분의 C++의 기본 타입(primitive type)은 추가 코드없이 shiboken에 의해 처리됩니다.

다음으로, 위에서 언급한 두 가지 클래스를 선언합니다. 그 중 하나는 "객체-타입(object-type)"이고 다른 하나는 "값-타입(value-type)" 입니다.

주된 차이점은 생성된 코드에서 '객체-타입'이 포인터(pointer)로 전달되는 반면, '값-타입'은 복사된다는 것입니다(value semantics).

타입시스템 파일에서 클래스의 이름을 지정하면, shiboken이 클래스에 선언된 모든 메소드에 대한 바인딩을 자동으로 생성하므로 모든 메소드 이름을 수동으로 언급할 필요가 없습니다...

어떻게든 기능을 수정하고 싶지 않다면. 그점은 우리를 다음 주제로 이끌 것입니다: '소유권 규칙(ownership rules)'.

Shiboken은 마술처럼 파이썬 코드에 할당된 C++ 객체를 해제할 책임이 있는 누구에게 있는지 알 수는 없습니다. 소유권을 추측할 수는 있지만, 항상 정확한 추측은 아닙니다.

많은 케이스가 있을 수 있습니다. Python 객체의 참조 카운트(ref count)가 0이 되면 Python은 C++ 메모리를 해제(release)해야 합니다. 또는 Python은 C++ 라이브러리 내부의 어떤 지점에서 삭제될 것이라는 가정 하에, C++ 객체를 절대로 삭제하면 안됩니다. 또는 QWidgets와 같은 다른 객체를 부모(parent)로 사용했을 수도 있습니다.

우리의 경우 clone() 메서드는 C++ 라이브러리 내에서만 호출되며, C++ 코드가 복제된 객체(cloned object)를 릴리스할 것(releasing)이라고 가정합니다.

addIcecreamFlavor()에 관해서는, 트럭이 Icecream 객체를 소유하고(owns) 있고, 트럭이 파괴되면 제거된다는 것을 알고 있습니다. 따라서 소유권(ownership)은 "C++"로 다시 설정됩니다.

소유권 규칙(ownership rule)을 지정하지 않으면, 이 경우 Python 이름이 범위를 벗어날 때 C++ 객체가 삭제됩니다.

## 빌딩(Building)

Universe 커스텀 라이브러리를 작성한 다음, 바인딩을 생성하기 위해 잘 정리된 대부분의 일반 CMakeLists.txt 파일을 제공합니다. 이 파일은 사용자 고유의 라이브러리에 재사용할 수 있습니다.

주로 "cmake."를 호출하여 프로젝트를 구성한 다음, 원하는 툴체인을 사용하여 빌드합니다 ('(N)Makefiles' 제네레이터를 사용하는 것을 권장합니다.)

프로젝트를 빌드한 결과 libuniverse(so/dylib/dll)와 Universe(so/pyd)의 두 공유 라이브러리가 생깁니다.

전자는 커스텀한 C++ 라이브러리이고, 후자는 Python 스크립트에서 가져올 수있는 파이썬 모듈입니다.

물론 shiboken(파이썬 바인딩 생성을 위해 생성된 .h/.cpp 파일)에 의해 생성된 중간 파일도 있습니다. 왜 무언가가 컴파일에 실패했는지 또는 왜 그렇게 행동하지 않는지 디버깅할 필요가 없다면 걱정하지 마십시오. 버그 리포트를 보내 주시면 감사하겠습니다!

더 자세한 빌드 지침과 주의해야 할 사항(예: Windows)은 README.md 파일 예에서 찾을 수 있습니다.

마지막으로 파이썬 부분을 살펴 보겠습니다.

## 파이썬 모듈 사용하기

다음의 작은 스크립트는 Universe 모듈을 사용하고, Icecream에서 파생되며, 가상 메서드(virtual method)를 구현하고, 객체(object)를 인스턴스화하는 등의 작업을 수행합니다.

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

우리 모듈에서 클래스를 가져온 후에 우리는 "맛(flavours)"을 커스텀하게 정의한 두 가지 Icecream 타입을 생성합니다.

그런 다음 트럭(truck)을 만들고, 보통맛을 내는 아이스크림을 추가하고, 특별한(special) 맛을 낸 아이스크림을 두 개 추가합니다.

이제 우리는 아이스크림을 배달하려고 합니다.

배달이 실패하면, 이전의 맛을 복사한 새로운 트럭과, 모든 고객을 만족시킬 특별한 마법의(*magical*) 맛을 만듭니다.

위의 스크립트는 C++ 타입에서 파생된 사용법, 오버라이딩(overriding)된 가상 메소드(virtual method), 객체 생성 및 파기 등을 간략하게 보여줍니다.

위에서 언급했듯이 전체 소스 및 추가 빌드 지침은 pyside-setup/examples/samplebinding의 프로젝트 저장소에서 찾을 수 있습니다.

이 작은 소개가 여러분에게 Shiboken의 힘과 파이썬을 위한 Qt를 만드는 방법, 그리고 어떻게 할 수 있었는지를 보여주기를 바랍니다.

해피 바인딩(Happy binding)!

## 글

[글 목록](README.md)

