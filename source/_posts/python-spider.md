---
title: python-spider
date: 2019-08-06 10:53:56
tags:
 - python 
categories: python
---

## requests
获取网页
response = requests.post(url, headers)
返回的response包含有网页返回的内容。
response.text以文字方式访问。

## beautiful soup
soup.find_all()

## selenium
1. xpath查找
查找class为"wos-style-checkbox"，type为"checkbox"的任意elements。可以把\*号换成input，就变成查找满足上述条件的input elemetns。
``` python
driver.find_elements_by_xpath("//*[@type='checkbox'][@class='wos-style-checkbox']")
```
2. 访问元素的内容
element.text
3. 复选框选中与取消选中
``` python
checkbox = driver.find_element_by_id("checkbox")
if checkbox.is_selected():  #如果被选中
    checkbox.click()    # 再点击一次，就变成了取消选中
else:   # 如果没有被选中
    checkbox.click()    # 再点击一次，就变成了选中
```
4. 提交表单
``` python
keywords = driver.find_element_by_id("input")
# 清空表单默认值
keywords.clear()
# 提交表单
keywords.send_keys(values)
```
5. 输完表单内容需要回车
直接在表单内容中添加\n即可。
6. 下拉框
使用Select对象
```
from selenium.webdriver.support.ui import Select
from selenium import webdriver
s = Select(driver.find_element_by_id("databases"))

s.select_by_value("value")
s.select_by_index(index)
s.select_by_visible_text("visible_text")
```
7. 模拟鼠标操作
```
from selenium.webdriver.common.action_chains import ActionChains

# 定位element
arrow = driver.find_element_by_id("next")
# 单击
ActionChains(driver).click(arrow).perform()
# 双击
ActionChains(driver).double_click(arrow).perform()
```
8. display:none和visible: hidden
display:none表示完全不可见，不占据页面空间，而visible: hidden仅仅隐藏了element的显示效果，仍然占据页面空间，并且是可以被定位到，但是无法被访问（如selenium的click,clear以及send_keys等，会报错ElementNotVisibleException': Message: Element is not currently visible and so may not be interacted with）。



## 参考文献
1.https://www.w3schools.com/xml/xml_xpath.asp
2.https://devhints.io/xpath
3.https://zhuanlan.zhihu.com/p/31604356
