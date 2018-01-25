---
title: PyQt5使用QStackedwidget
date: 2017-08-08 17:14:14
tags: PyQt5
categories: python
---
QListWidget和QStackedWidget在ui文件中.

```python
# -*- coding:utf-8 -*-
import os
import functools
from PyQt5 import uic
from PyQt5 import QtWidgets
from utils import loadJson
from page1 import ControlPanel
from utils import loadJson


class ControlPanel2(QtWidgets.QWidget):

    def __init__(self, parent=None):
        super(ControlPanel2, self).__init__(parent)
        self._config = loadJson()
        self.ctrlpanelWidth = self._config['controlpanel']['width']
        self.ctrlpanelHeight = self._config['controlpanel']['height']
        self.ctrl_panel = ControlPanel()
        self.initUI()
        
    def initUI(self):
        self.ui = uic.loadUi(os.path.join(
            os.path.dirname(__file__), "res/ctrlpanel.ui"), self)
        self.setFixedSize(self.ctrlpanelWidth, self.ctrlpanelHeight)
        for i, key in enumerate(sorted(self._config["panels"].keys())):
            item = QtWidgets.QListWidgetItem(self.ui.panellist)
            item.setText(key)
            self.ui.panellist.insertItem(i, item)
            self.ui.stackedWidget.insertWidget(i, QtWidgets.QLabel("{}".format(key)))  
        # 设置qss
        self.setStyleSheet(
            "QListWidget::item{width: 65px; height: 35px; text-align: center;}")
        self.ui.panellist.setCurrentRow(0)
        self.ui.stackedWidget.setCurrentIndex(0)
        self.ui.panellist.currentRowChanged.connect(self.on_showPage)

    def on_showPage(self, index):
        # index = self.ui.panellist.currentIndex()
        print("current index =", index)
        self.ui.stackedWidget.setCurrentIndex(index)
```
