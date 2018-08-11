
## 파이썬 상호 작용을 위한 QML과 Qt

![](https://j2doll.github.io/Qt-for-Python-Docs-Kor/image/QtForPython.png)

- 원문:  [http://blog.qt.io/blog/2018/05/14/qml-qt-python/](http://blog.qt.io/blog/2018/05/14/qml-qt-python/)

데스크톱 애플리케이션용 QtWidgets 외에도 Qt에는 또 다른 UI 기술인 QML이 있습니다.

오늘 저는 QML이 [Qt for Python](http://www.pyside.org/)과 상호작용하는 방법을 보여 드리고자 합니다. 대략적으로 튜토리얼 [declarative/extending/chapter3-bindings](https://doc.qt.io/qt-5.10/qtqml-tutorials-extending-qml-example.html#chapter-3-adding-property-bindings)을 기반으로 합니다.

먼저 .qml 파일을 로드하기 위한 일반적인 보일러 플레이트(boiler plate) 코드를 살펴 보겠습니다. 우리는 QGuiApplication과 QQuickView를 인스턴스화 합니다.
 QML 요소(element)가 뷰 크기(View Size)로 확장되도록 크기 조정 모드 속성을 SizeRootObjectToView로 설정합니다.

```python
app = QGuiApplication(sys.argv)
view = QQuickView()
view.setResizeMode(QQuickView.SizeRootObjectToView)
```

그런 다음 setSource 메소드를 사용하여 뷰에 QML 파일을 로드하도록 지시합니다.
QML 파일은 일반적으로 Python 스크립트 옆에 있으므로, os 모듈의 함수를 사용하여 완전한 경로를 만듭니다.

```python
current_path = os.path.abspath(os.path.dirname(__file__))
qml_file = os.path.join(current_path, 'app.qml')
view.setSource(QUrl.fromLocalFile(qmlFile))
if view.status() == QQuickView.Error:
    sys.exit(-1)
```
이제 보기가 표시되고 응용 프로그램이 실행됩니다. 올바른 순서를 지키려면 응용 프로그램을 종료하기 전에 뷰(view) 오브젝트에서 del을 호출해야 합니다.

```python
view.show()
res = app.exec_()
del view
sys.exit(res)
```

이 코드를 사용하여 QML 파일을 표시할 수 있습니다. 예를 들어 app.qml을 만들 때 다음과 같이 최소한의 hello 영역이 표시됩니다.

```qml
import QtQuick 2.0

Text {
    text : 'Hello, world!'
}
```

### 파이썬으로 작성된 클래스를 사용하여 QML 확장하기

QQuickPaintedItem 위에 뭔가를 구현해 보겠습니다.

```python
class PieChart (QQuickPaintedItem):
    def __init__(self, parent = None):
        QQuickPaintedItem.__init__(self, parent)
        self.color = QColor()

    def paint(self, painter):
        pen = QPen(self.color, 2)
        painter.setPen(pen);
        painter.setRenderHints(QPainter.Antialiasing, True);
        # From drawPie(const QRect &rect, int startAngle, int spanAngle)
        painter.drawPie(self.boundingRect().adjusted(1,1,-1,-1),
            90 * 16, 290 * 16);

    def getColor(self):
        return self.color

    def setColor(self, value):
        if value != self.color:
            self.color = value
            self.update()
            self.colorChanged.emit()

    colorChanged = Signal()
    color = Property(QColor, getColor, setColor, notify=colorChanged)
```

QQuickPaintedItem.paint() 메서드를 재정의하여 간단한 원형 차트를 그립니다. 색상은 Qt에 노출되도록 속성으로 정의됩니다. 유형을 QML에 알리기 위해 다음을 삽입합니다.

```qml
qmlRegisterType(PieChart, 'Charts', 1, 0, 'PieChart');
```
QGuiApplication을 생성한 후.

app.qml은 다음과 같이 변경할 수 있습니다.

```qml
import Charts 1.0
import QtQuick 2.0

Item {
    width: 300; height: 200

    PieChart {
        id: chartA
        width: 100; height: 100
        color: "red"
        anchors.centerIn:parent
    }

    MouseArea {
        anchors.fill: parent
        onClicked: { chartA.color = "blue" }
    }

    Text {
        anchors {
            bottom: parent.bottom;
            horizontalCenter: parent.horizontalCenter;
            bottomMargin: 20
        }
        text: "Click anywhere to change the chart color"
    }
}
```
맞춤 유형이 표시됩니다.

또한 마우스를 클릭하면 color 속성이 변경됩니다.

## 글

[글 목록](README.md)
