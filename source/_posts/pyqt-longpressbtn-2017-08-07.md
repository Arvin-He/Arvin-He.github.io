---
title: PyQt5长按按钮循环执行命令
date: 2017-08-07 17:39:12
tags: PyQt5
categories: python
---

```python
# -*- coding:utf-8 -*-
import os
import time
import functools
from PyQt5 import uic
from PyQt5 import QtWidgets
from utils import loadJson
from punggol_rpc import punggol_exec


class ControlPanel(QtWidgets.QWidget):

    def __init__(self, parent=None):
        super(ControlPanel, self).__init__(parent)
        self._config = loadJson()
        self.ctrlpanelWidth = self._config['controlpanel']['width']
        self.ctrlpanelHeight = self._config['controlpanel']['height']
        self.initUI()

    def initUI(self):
        self.ui = uic.loadUi(os.path.join(
            os.path.dirname(__file__), "res/ctrlpanel.ui"), self)
        self.setFixedSize(self.ctrlpanelWidth, self.ctrlpanelHeight)
        for attr in dir(self.ui):
            obj = getattr(self.ui, attr)
            if isinstance(obj, QtWidgets.QPushButton) or \
               isinstance(obj, QtWidgets.QToolButton):
                obj.setAutoRepeat(True)
                obj._repeate = False
                obj.clicked.connect(
                    functools.partial(self.on_handleClicked, obj))

    def on_handleClicked(self, btn):
        if btn.isDown():
            if btn._repeate is False:
                btn._repeate = True
                btn.setAutoRepeatInterval(50)
            else:
                self.on_pressed(btn)
        elif btn._repeate is True:
            btn._repeate = False
            self.on_released(btn)
        else:
            self.on_clicked(btn)

    def on_clicked(self, btn):
        try:
            if btn.property("clicked_cmd") is not None:
                punggol_exec(btn.property("clicked_cmd"))
        except Exception as e:
            print(e)
            return

    def on_pressed(self, btn):
        try:
            if btn.property("pressed_cmd") is not None:
                punggol_exec(btn.property("pressed_cmd"))
        except Exception as e:
            print(e)
            return
    
    def on_released(self, btn):
        try:
            if btn.property("released_cmd") is not None:
                punggol_exec(btn.property("released_cmd"))
        except Exception as e:
            print(e)
            return
```
