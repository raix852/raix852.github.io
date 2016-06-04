---
title: Python - @property
date: 2016-06-04 21:38:52
tags: [python]
---

今天在 trace 一個 python 專案時，看到類似 getter/setter 函式，不過實際使用時卻不像其他語言的 getter/setter 用法。於是就去 google 一下，找到 @property 的用法。

原來，在 python class 中，因為沒有 private 變數，全部都是 public ，任何人都可以存取，頂多只有透過命名風格在變數前加 '\_' 或 '\_\_' ，讓別人了解這個變數是給內部使用，不建議直接呼叫。也因此 getter/setter 的設計模式在 python 較沒意義，取而代之的是使用 @property 裝飾器(decorator)來達到類似的效果。

### 範例 1
```python
#!/usr/bin/python

class Example(object):
    _text = 'This is an origin string'

    @property
    def text(self):
        return self._text

    @text.setter
    def text(self, string):
        if isinstance(string, str):
            self._text = string
        else:
            raise TypeError('%s url must be str, got %s:' % (type(self).__name__,
                type(url).__name__))


if __name__ == '__main__':
    ex = Example()
    string = 'This is a test'

    print 'Get Example.text: %s' % (ex.text)

    print 'Set Example.text: %s' % (string)
    ex.text = string

    print 'Get Example.text: %s' % (ex.text)

```

### 執行結果
```sh
$ ./decorator_property.py
Get Example.text: This is an origin string
Set Example.text: This is a test
Get Example.text: This is a test
```

不過在 trace 的 python 專案中，用的是另一種方法。

### 範例 2
```python
#!/usr/bin/python

class Example(object):
    _text = 'This is an origin string'

    def _get_text(self):
        return self._text

    def _set_text(self, string):
        if isinstance(string, str):
            self._text = string
        else:
            raise TypeError('%s url must be str, got %s:' % (type(self).__name__,
                type(url).__name__))

    text = property(_get_text, _set_text)

if __name__ == '__main__':
    ex = Example()
    string = 'This is a test'

    print 'Get Example.text: %s' % (ex.text)

    print 'Set Example.text: %s' % (string)
    ex.text = string

    print 'Get Example.text: %s' % (ex.text)
```

### 執行結果
```sh
$ ./decorator_property_2.py
Get Example.text: This is an origin string
Set Example.text: This is a test
Get Example.text: This is a test
```

可以看到兩種寫法都可以達到相同的效果，而且用 `type(ex.text)` 看型態都是 `<type 'str'>` 。

至於哪種寫法比較好？就本人覺得，比較偏好範例 2 ，畢竟可以對應到其他語言的寫法，相較之下可讀性較高。


## Link
1. [使用@property](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386820062641f3bcc60a4b164f8d91df476445697b9e000)
2. [Python @property versus getters and setters](http://stackoverflow.com/questions/6618002/python-property-versus-getters-and-setters)
