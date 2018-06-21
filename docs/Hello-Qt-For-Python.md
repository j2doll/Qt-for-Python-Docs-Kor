
## 헬로우 큐트 포 파이썬 (Hello Qt for Python)

![](https://j2doll.github.io/Qt-for-Python-Docs-Kor/pysidelogo.png)

- 원문:  [http://blog.qt.io/blog/2018/05/04/hello-qt-for-python/](http://blog.qt.io/blog/2018/05/04/hello-qt-for-python/)

- 번역:  [https://j2doll.github.io/Qt-for-Python-Docs-Kor/](https://j2doll.github.io/Qt-for-Python-Docs-Kor/)

첫 번째 Qt for Python의 기술 프리뷰 릴리스가 여기 있습니다!

그래서 파이썬 세계에 문을 여는 방법에 대한 간단한 예제를 제공하고자 합니다.

일단 QWidgets을 사용하여 Python용 Qt의 단순성을 보여주는 간단한 애플리케이션을 작성해 보겠습니다.

모든 스크립트의 구조는 다음과 같습니다:

일단 QApplication을 생성합니다.

그리고 나서 모든 QWidgets과 사용하려는 구조체를 포함합니다. (예> QLabel 등)

어플리케이션을 보여 주면서 QApplication를 시작합니다.
이런 아이디어를 하나로 합치면 다음과 같이됩니다.

```python
# hello_world.py
from PySide2.QtWidgets import QApplication, QLabel

app = QApplication([])
label = QLabel("Hello Qt for Python!")
label.show()
app.exec_()
```

이를 실행하기 위해 간단한 python hello_world.py가 작업을 수행합니다.

이때 진짜 질문은 다음과 같습니다: Qt 클래스(class)의 메소드(method)에 액세스하는 방법은 무엇인가?

프로세스를 단순화하기 위해 Qt API를 유지합니다. 예를 들어, QLabel의 크기(size)를 지정하려면 C++에서 다음과 같이 됩니다.

```cpp
QLabel *label = new QLabel();
label->setText("Hello World!");
label->resize(800, 600);
```

Python에서 Qt를 사용하는 방법은 다음과 같습니다:

```python
label = QLabel()
label.setText("Hello World!")
label.resize(800, 600)
```

이제 C++와 동등하게 적용하는 방법을 알게 되었으므로, 보다 정교한 응용 프로그램을 작성할 수 있습니다.

```python
import sys
import random
from PySide2.QtCore import Qt
from PySide2.QtWidgets import (QApplication, QWidget,
    QPushButton, QLabel, QVBoxLayout)


class MyWidget(QWidget):
    def __init__(self):
        QWidget.__init__(self)

        self.hello = ["Hallo welt!", "Ciao mondo!",
            "Hei maailma!", "Hola mundo!", "Hei verden!"]

        self.button = QPushButton("Click me!")
        self.text = QLabel("Hello World")
        self.text.setAlignment(Qt.AlignCenter)

        self.layout = QVBoxLayout()
        self.layout.addWidget(self.text)
        self.layout.addWidget(self.button)
        self.setLayout(self.layout)

        self.button.clicked.connect(self.magic)

    def magic(self):
        self.text.setText(random.choice(self.hello))


if __name__ == "__main__":
    app = QApplication(sys.argv)
    widget = MyWidget()
    widget.resize(800, 600)
    widget.show()
    sys.exit(app.exec_())
```

 Qt 개발에 익숙하지 않다면, 특정 클래스를 확장하여 필요에 맞게 수정하는 것이 일반적입니다.

앞의 예제에서는 QWidget을 기본 클래스(base class)로 사용하고, QLabel과 QPushButton을 포함했습니다.

예제 응용 프로그램은 정말 간단합니다:

먼저 Hello World를 포함하는 목록을 다국어로 작성합니다.

그런 다음 QPushButton과 QLabel을 특정 정렬, 글꼴 및 크기로 초기화합니다.

그 다음, 우리는 객체를 포함하기 위해 QVBoxLayout을 생성하고 그것을 우리의 클래스에 할당합니다.

마지막으로 QPushButton의 clicked() 시그널(signal)을 magic이라는 메서드(method)에 연결합니다.

결과적으로 버튼을 누를 때마다 Hello World가 임의의 언어로 표시됩니다!

이 간단한 스크립트의 구조는 Qt for Python을 사용하는 대부분의 응용 프로그램의 기본이 될 것이므로, Qt for Python이 발표되자 마자 테스트해 보시기 바랍니다!

## 글

[글 목록](README.md)
