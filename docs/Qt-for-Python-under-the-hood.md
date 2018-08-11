
## 큐트 포 파이썬: 내부를 들여다 보기 (Qt for Python: under the hood)

![](https://j2doll.github.io/Qt-for-Python-Docs-Kor/image/QtForPython.png)

- 원문: [http://blog.qt.io/blog/2018/05/24/qt-for-python-under-the-hood/](http://blog.qt.io/blog/2018/05/24/qt-for-python-under-the-hood/)

- 번역: [https://j2doll.github.io/Qt-for-Python-Docs-Kor/](https://j2doll.github.io/Qt-for-Python-Docs-Kor/)

오늘 저는 "Qt for Python bindings" 생성 프로세스에 대해 말씀 드리고자 합니다.

다음 다이어그램은 The Qt Company가 PySide2 모듈을 제공하기 위해 사용하고있는 내부 프로세스의 가장 일반적인 상호 작용을 보여줍니다:

![](https://j2doll.github.io/Qt-for-Python-Docs-Kor/image/qtforpython-underthehood.png)

PySide 프로젝트가 2009 년에 시작되었을 때 팀은 외부 도구를 사용하여 Qt C++ 헤더(header)에서 Python 바인딩(binding)을 생성하기로 결정했습니다.

주된 관심사 중 하나는 모든 Qt C++ 구문을 올바르게 처리하는 도구를 사용하는 것 외에도 최종 패키지의 크기였습니다.

이전 선택은 과도하게 템플리트(template)를 사용하므로 다른 대안이 필요했습니다.

팀은 몇 가지 다른 옵션을 분석한 후, 생성기인 Shiboken을 직접 작성하기로 결정했습니다.

Shiboken은 Qt 헤더(header)를 파싱하고 Clang을 사용하여 모든 Qt 클래스(class)의 정보를 가져 오는 ApiExtractor 라이브러리를 사용합니다.
(ApiExtractor 라이브러리는 QtScriptGenerator라는 이전 프로젝트를 기반으로 합니다.)

이 단계는 Qt에 의존하지 않으므로 다른 C++ 프로젝트에도 사용할 수 있습니다.

또한 Shiboken은 획득된 정보를 수정하여 C++ 클래스를 Python 세계로 적절하게 표현하고 조작할 수 있는 Typesystem(XML 기반)을 보유하고 있습니다.

이 Typesystem을 통해 메소드(method)를 제거하고 특정 클래스에 추가할 수 있으며 각 함수의 인수를 수정할 수도 있습니다.
이는 데이터 구조나 유형을 적절히 처리하기 위한 C++ 및 Python 충돌과 의사 결정에 필수적입니다.

이 프로세스의 결과는 CPython으로 작성된 래퍼(wrapper) 세트(set)입니다. 이 래퍼를 사용하면 Python2라는 Python 모듈을 쉽게 컴파일하고 제공할 수 있습니다.

ApiExtractor 및 Shiboken과 관련된 공식 문서는 공식 Qt for Python 문서에 포함되어 개발에 참여할 수 있으며, Shiboken이 어떻게 작동하는지 궁금하다면 향후 블로그(blog.qt.io) 게시물을 계속 지켜봐 주십시오!

## 글

[글 목록](README.md)
