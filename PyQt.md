# 主窗口类型
- QMainWindow: 可以包含菜单栏、工具栏、状态栏和标题栏，是最常见的窗口形式
- QDialog: 是对话窗口的基类。没有菜单栏、工具栏、状态栏
- QWidget: 不确定窗口的用途时使用

## 窗口中的方法()
继承QMainWindow
- self.setWindowTitle('设置窗口的标题')
- self.resize(width, height)  设置窗口的尺寸
- self.status = self.statusBar().showMessage('消息的内容', 存在的时间)
- size = self.geometry()  获取窗口坐标系
- self.setGemoetry(width, height, x, y)  设置窗口大小和位置

全局
- app.setWindowIcon(QIcon('图标路径'))  设置窗口图标
- QApplication().setWindowIcon('图标路径')  设置应用程序图标
- screen = QDesktopWidget().screenGeometry()  获取屏幕坐标系
- screen.width()  screen.height()
# 控件

## PushButton

```python
button = QPushButton('按钮上要显示的文字/热键')
button.setText('按钮上显示的文字')
# 将按钮设置为二值类
button.setCheckable(True)
button.toggle()

# 传参式绑定槽
button.clicked.connect(lambda:self.which_button(button))

# 在文本前面显示图像
button2 = QPushButton('图像按钮')
button2.setIcon(QIcon(QPixmap('图像路径')))

# 设置按钮不可用
button3 = QPushButton('不可用按钮')
button3.setEnable(False)

# 设置默认按钮
button4 = QPushButton('&MyButton')
button4.setDefault(True)

def which_button(self, btn):
    print(btn.text())
```

## RadioButton

默认未选中

```python
layout = QHBoxLayout()

radio_button = QRadioButton('单选按钮1')
# 设置单选按钮默认是否被选中
radio_button.setChecked(True)
radio_button.toggled.connect(self.radio_status)
layout.addWidget(radio_button)

radio_button2 = QRadioButton('单选按钮2')
radio_button2.toggled.connect(self.radio_status)
layout.addWidget(radio_button2)

def radio_status(self):
    radio_button = self.sender()
    if radio_button.isChecked():
        print(radio_button.text() + '被选中')
	else:
		print(radio_button.text() + '未被选中')
```

## CheckBox

3种状态：

- 未选中(默认)：0
- 半选中：1
- 选中：2

```python
layout = QHBoxLayout

check_box = QCheckBox('未选中复选框1')
check_box2 = QCheckBox('选中复选框2')
check_box2.setChecked(True)
check_box3 = QCheckBox('半选中复选框3')
check_box3.setTristate(True)
check_box3.setCheckState(Qt.PartiallyChecked)

layout.addWidget(check_box)
layout.addWidget(check_box2)
layout.addWidget(check_box3)
```

## ComboBox

下拉列表控件

```python
layout = QVBoxLayout()

self.label = QLable('请选择编程语言')

self.cb = QComboBox()
self.cb.addItem('C++')
self.cb.addItem('Python')
self.cb.addItem(['Java', 'C#'])

self.cb.currentIndexChanged.connect(self.selection_change)

layout.addWidget(self.label)
layout.addWidget(self.cb)
self.setLayout(layout)

def selection_change(self, i):
    self.label.setText(self.cb.currentText())
    self.label.adjustSize()
    
    for count in range(self.cb.count()):
        print('item' + str(count) + '=' + self.cb.itemText(count))
        print('current index', i, 'selection changed', self.cb.currentText())
```

## Slider

滑块控件

```python
layout = QVBoxLayout()

self.label = QLable('qt5')
self.label.setAlignment(Qt.AlignCenter)
layout.addWidget(self.label)

# 垂直的滑块
self.slider2 = QSlider(Qt.Vertical)
# 水平的滑块
self.slider = QSlider(Qt.Horizontal)
# 设置最小值
self.slider.setMinimum(12)
# 设置最大值
self.slider.setMaximum(48)
# 设置步长
self.slider.setSingleStep(3)
# 设置当前值
self.silder.setValue(18)
# 设置刻度的位置 下方
self.slider.setTickPosition(QSlider.TicksBelow)
# 设置刻度的间隔
self.slider.setTickInterval(6)
layout.addWidget(self.slider)

self.slider.valueChanged.connect(self.value_change)

self.setLayout(layout)

def value_change(self):
    c_value = self.sender().value()
    print('当前值为：%s' % c_value)
    self.label.setFont(QFont('Arial', c_value))
```



## Label

setAlignment()  设置文本的对齐方式

setIndent()  设置文本缩进

text()  获取文本内容

setBuddy()  设置伙伴关系

setText()  设置文本内容，可根据html标签个性化设置

selectedText()  返回所选的字符

setWordWrap()  设置是否允许换行

常用的信号(事件)：

- linkHovered  鼠标滑过时触发
- linkActivate  鼠标单击时触发

```python
from PyQt5.QtGui import QPalette, QPixmap
from PyQt5.QtCore import Qt

label1 = Qlabel()
label1.setText('<font color=yellow>这是一个文本标签</font>')
label1.setAutoFillBackground(True)
# 创建调色板
palette = QPalette()
palette.setColor(QPalette.Window, Qt.blue)
# 添加颜色
label1.setPalette(palette)
# 设置文本对齐方式
label1.setAlignment(Qt.AlignCenter)


label2 = Qlabel()
# 设置链接
label2.setOpenExternallinks(True)
label2.setText("<a href='https://bilibili.com'>跳转到B站</a>")


label3 = Qlabel()
label3.setAlignment(Qt.AlignCenter)
label3.setToolTip('这是Label3')
# 设置图片
label3.setPixmap(QPixmap('图片路径'))
```

```python
import sys
from PyQt5.QtWidgets import *

"""
网格布局的参数
main_layout = QGridLayout(self).addWidget(控件对象, rowIndex, columnIndex, rowTakes, columnTakes)
"""

class QLabelBuddy(QDialog):
    def __init__(self):
        super().__init__()
        self.init_ui()

    def init_ui(self):
        self.setWindowTitle('QLabel伙伴关系')

        # 设置热键 ALT+N
        name_label = QLabel('&Name')
        name_line_edit = QLineEdit()
        # 设置伙伴关系
        name_label.setBuddy(name_line_edit)

        pwd_label = QLabel('&Password')
        pwd_line_edit = QLineEdit()
        # 设置伙伴关系
        pwd_label.setBuddy(pwd_line_edit)

        btn_ok = QPushButton('&OK')
        btn_cancel = QPushButton('&Cancel')

        main_layout = QGridLayout(self)
        main_layout.addWidget(name_label, 0, 0)
        main_layout.addWidget(name_line_edit, 0, 1, 1, 2)

        main_layout.addWidget(pwd_label, 1, 0)
        main_layout.addWidget(pwd_line_edit, 1, 1, 1, 2)

        main_layout.addWidget(btn_ok, 2, 1)
        main_layout.addWidget(btn_cancel, 2, 2)


if __name__ == '__main__':
    app = QApplication(sys.argv)
    mainWindow = QLabelBuddy()
    mainWindow.show()
    sys.exit(app.exec_())
```

## LineEdit

4种回显模式

- Normal
- NoEcho
- Password
- PasswordEchoOnEdit

```python
# 创建表单布局
form_layout = QFormLayout()

normal_line_edit = QLineEdit()
no_echo_line_edit = QLineEdit()
passowrd_line_edit = QLineEdit()
passowrd_echo_on_edit_line_edit = QLineEdit()

form_layout.addRow('Normal', normal_line_edit)
form_layout.addRow('NoEcho', no_echo_line_edit)
form_layout.addRow('Password', passowrd_line_edit)
form_layout.addRow('PasswordEchoOnEdit', passowrd_echo_on_edit_line_edit)

# 灰色提示 输入时消失
normal_line_edit.setPlaceholderText('this one is normal')

# 设置回显模式
normal_line_edit.setEchoMode(QLineEdit.Normal)

self.setLayout(form_layout)
```

用校验器限制LineEdit输入的类型

```python
from PyQt5.QtGui import QIntValidator, QDoubleValidator, QRegExpValidator
from PyQt5.QtCore import QRegExp

# 创建表单布局
form_layout = QFormLayout()

int_line_edit = QLineEdit()
double_line_edit = QLineEdit()
regex_line_edit = QLineEdit()

form_layout.addRow('整数类型', int_line_edit)
form_layout.addRow('浮点数类型', double_line_edit)
form_layout.addRow('正则', regex_line_edit)

int_line_edit.setPlaceholderText('只能输入整数')
double_line_edit.setPlaceholderText('只能输入浮点数')
regex_line_edit.setPlaceholderText('只能输入符合正则的字符')

# 创建整型校验器 1~99
int_validator = QIntValidator(self)
int_validator.setRange(1, 99)

# 创建浮点数校验器 -360~360
double_validator = QDoubleValidator(self)
double_validator.setRange(-360, 360)
double_validator.setNotation(QDoubleValidator.StandardNotation)
# 设置精度 小数点后两位
double_validator.setDecimals(2)

# 创建正则校验器 字符和数字
reg = QRegExp('^[a-zA-Z0-9]+$')
reg_validator = QRegExpValidator(self)
reg_validator.setRegExp(reg)

# 设置校验器
int_line_edit.setValidator(int_validator)
# 设置最大长度
int_line_edit.setMaxLength(2)
# 设置字体
int_line_edit.setFont(QFont('Arial', 20))
double_line_edit.setValidator(double_validator)
regex_line_edit.setValidator(reg_validator)

self.setLayout(form_layout)
```

用掩码限制LineEdit输入的类型（网上找）

## TextEdit

```python
import sys
from PyQt5.QtWidgets import *

class TextEditDemo(QWidget):
    def __init__(self):
        super().__init__()
        self.init_ui()

    def init_ui(self):
        self.setWindowTitle('QTextEdit控件演示')

        self.setGeometry(300, 200, 500, 400)

        self.text_edit = QTextEdit()
        self.button_text = QPushButton('显示文本')
        self.button_html = QPushButton('显示HTML')
        self.button_to_text = QPushButton('转化为文本')
        self.button_to_html = QPushButton('转化为HTML')

        layout = QVBoxLayout()

        layout.addWidget(self.text_edit)
        layout.addWidget(self.button_text)
        layout.addWidget(self.button_html)
        layout.addWidget(self.button_to_text)
        layout.addWidget(self.button_to_html)

        self.setLayout(layout)

        self.button_text.clicked.connect(self.on_click_text)
        self.button_html.clicked.connect(self.on_click_html)
        self.button_to_text.clicked.connect(self.on_click_to_text)
        self.button_to_html.clicked.connect(self.on_click_to_html)

    def on_click_text(self):
        self.text_edit.setPlainText('Hellow World')

    def on_click_html(self):
        self.text_edit.setHtml('<font color="skyblue"> Hellow World aaa </font>')

    def on_click_to_text(self):
        self.text_edit.setPlainText(self.text_edit.toPlainText())

    def on_click_to_html(self):
        self.text_edit.setHtml('<font color="red">' + self.text_edit.toHtml() + "</font>")


if __name__ == '__main__':
    app = QApplication(sys.argv)
    mainWindow = TextEditDemo()
    mainWindow.show()
    sys.exit(app.exec_())
```

## 计数器控件

```python
layout = QVBoxLayout()

self.label = QLabel('当前值')
self.label.setAlignment(Qt.AlignCenter)
layout.addWidget(self.label)

self.sb = QSpinBox()
# 设置范围
self.sb.setRange(10, 100)
# 设置步长
self.sb.setSingleStep(3)
layout.addWidget(self.sb)

self.sb.valueChange.connect(self.value_change)

def value_change(self):
    self.label.setText('当前值' + str(self.sb.value())
```

## Dialog

对话框

```python
dialog = QDialog()
dialog.setWindowTitle('对话框')
# 设置对话框为应用模式 即对话框弹出状态下无法操作母窗口
dialog.setWindowModality(Qt.ApplicationModal)
```

### 消息对话框

类型：

- 关于对话框 QMessageBox.about
- 错误对话框 QMessageBox.critical
- 警告对话框 QMessageBox.warning
- 提问对话框 QMessageBox.question
- 消息对话框 QMessageBox.information

差异：

- 显示的对话框的图标可能不相同
- 显示的按钮不一样

```python
layout = QVBoxLayout()

button1 = QPushButton()
button1.setText('关于对话框')
button1.clicked.connect(self.show_dialog)

button2 = QPushButton()
button2.setText('消息对话框')
button2.clicked.connect(self.show_dialog)

button3 = QPushButton()
button3.setText('警告对话框')
button3.clicked.connect(self.show_dialog)

button4 = QPushButton()
button4.setText('错误对话框')
button4.clicked.connect(self.show_dialog)

button5 = QPushButton()
button5.setText('提问对话框')
button5.clicked.connect(self.show_dialog)

layout.addWidget(button1)
layout.addWidget(button2)
layout.addWidget(button3)
layout.addWidget(button4)
layout.addWidget(button5)
self.setLayout(layout)

def show_dialog(self):
    text = self.sender().text()
    if text == '关于对话框':
        QMessageBox.about(self, '对话框标题上的文字', '对话框上显示的文字')
     elif text == '消息对话框':
        reply = QMessageBox.information(self, '对话框标题上的文字', '对话框上显示的文字', QMessageBox.Yes | QMessageBox.No, QMessageBox.Yes)
        print(reply == QMessageBox.Yes)
	elif text == '警告对话框':
        QMessageBox.warning(self, '对话框标题上的文字', '对话框上显示的文字', QMessageBox.Yes | QMessageBox.No, QMessageBox.Yes)
	elif text == '错误对话框':
        QMessageBox.critical(self, '对话框标题上的文字', '对话框上显示的文字', QMessageBox.Yes | QMessageBox.No, QMessageBox.Yes)
	elif text == '提问对话框':
        QMessageBox.question(self, '对话框标题上的文字', '对话框上显示的文字', QMessageBox.Yes | QMessageBox.No, QMessageBox.Yes)
```

### 输入对话框

- QInputDialog.getItem
- QInputDialog.getText
- QInputDialog.getInt

```python
layout = QFormLayout()

button1 = QPushButton('获取列表种的选项')
button1.clicked.connect(self.get_item)
self.line_edit1 = QLineEdit()

button2 = QPushButton('获取字符串')
button2.clicked.connect(self.get_text)
self.line_edit2 = QLineEdit()

button3 = QPushButton('获取整数')
button3.clicked.connect(self.get_int)
self.line_edit3 = QLineEdit()

layout.addRow(button1, self.line_edit1)
layout.addRow(button2, self.line_edit2)
layout.addRow(button3, self.line_edit3)

self.setLayout(layout)

def get_item(self):
    items = ('C', 'C++', 'Ruby', 'Python', 'Java')
    item, ok = QInputDialog.getItem(self, '请选择编程语言', '语言列表', items)
    if ok and item:
        self.line_edit1.setText(item)
        
def get_text(self):
    text, ok = QInputDialog.getText(self, '文本输入框', '输入姓名')
    if ok and text:
        self.line_edit2.setText(text)

def get_int(self):
    num, ok = QInputDialog.getText(self, '整数输入框', '输入数字')
    if ok and num:
        self.line_edit3.setText(str(num))
```

### 字体对话框

QFontDialog.getFont()

```python
layout = QVBoxLayout()

font_button = QPushButton('选择字体')
font_button.clicked.connect(self.get_font)

self.font_label = QLabel('测试字体')
layout.addWidget(font_button)
layout.addWidget(self.font_label)

self.setLayout(layout)

def get_font(self):
    font, ok = QFontDialog.getFont()
    if ok:
        self.font_label.setFont(font)
```

### 颜色对话框

QColorDialog.getColor()

```python
layout = QVBoxLayout()

color_button = QPushButton('选择文字颜色')
color_button.clicked.connect(self.get_color)

color_button1 = QPushButton('选择背景颜色')
color_button1.clicked.connect(self.get_bg_color)

self.color_label = QLabel('测试颜色')
layout.addWidget(color_button)
layout.addWidget(self.color_label)

self.setLayout(layout)

def get_color(self):
    color, ok = QColorDialog.getColor()
    p = QPalette()
    p.setColor(QPalette.WindowText, color)
    self.color_label.setPalette(p)

def get_bg_color(self):
    color, ok = QColorDialog.getColor()
    p = QPalette()
    p.setColor(QPalette.Window, color)
    self.color_label.setAutoFillBackground(True)
    self.color_label.setPalette(p)
```

### 文件对话框

QFileDialog

```python
layout = QVBoxLayout()

button = QPushButton('加载图片')
button.clicked.connect(self.load_image)
self.img_label = QLabel()
layout.addWidget(button)
layout.addWidget(self.img_label)

button1 = QPushButton('加载文本文件')
button1.clicked.connect(self.load_text)
layout.addWidget(button1)

self.contents = QTextEdit()
layout.addWidget(self.contents)

self.setLayout(layout)

def load_img(self):
    fname, _ = QFileDialog.getOpenFileName(self, '打开文件', '.', '图像文件(*.jpg *.png)')
    self.img_label.setPixmap(QPixmap(fname))
    
def load_text(self):
    dialog = QFileDialog()
    dialog.setFileMode(QFileDialog.AnyFile)
    dialog.setFilter(QDir.Files)
    
    if dialog.exec():
        filenames = dialog.selectedFiles()
        with open(filenames[0], 'r', encoding='utf-8') as f:
            data = f.read()
            self.contents.setText(data)
```



# 信号槽

```python
import sys
from PyQt5.QtWidgets import QApplication, QMainWindow, QPushButton, QHBoxLayout, QWidget

class QuitApplication(QMainWindow):
    def __init__(self):
        super().__init__()
        self.resize(300, 200)
        self.setWindowTitle('点击按钮退出程序')

        # 添加按钮
        button1 = QPushButton('退出')
        # 绑定方法
        button1.clicked.connect(self.on_click)

        # 设置水平布局
        layout = QHBoxLayout()
        layout.addWidget(button1)

        main_frame = QWidget()
        main_frame.setLayout(layout)

        self.setCentralWidget(main_frame)

    # 自定义的槽
    def on_click(self):
        sender = self.sender()
        print(sender.text())
        app = QApplication.instance()
        app.quit()


if __name__ == '__main__':
    app = QApplication(sys.argv)
    mainWindow = QuitApplication()
    mainWindow.show()
    sys.exit(app.exec_())
```

# 控件提示信息

```python
button1 = QPushButton('退出')
button1.setToolTip('别看了这是提示信息')
```

# 屏幕坐标系

```python
widget = QWidget()

widget.x()  # 窗口横坐标
widget.y()  # 窗口纵坐标
widget.width()  # 工作区宽度
widget.height()  # 工作区高度

widget.gemoetry().x()  # 工作区横坐标
widget.gemoetry().y()  # 工作区纵坐标
widget.gemoetry().width()  # 工作区宽度
widget.gemoetry().height()  # 工作区高度

widget.frameGemoetry().x()  # 窗口横坐标
widget.frameGemoetry().y()  # 窗口纵坐标
widget.frameGemoetry().width()  # 窗口宽度
widget.frameGemoetry().height()  # 窗口高度
```

