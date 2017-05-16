---
layout:     post
title:      pybabel国际化翻译
category: project
description: pybabel简单使用笔记

---


## 简介

TODO

## 第一部分：生成pot po mo

1 
创建babel.cfg

2 
修改需要自动变换的msg
```
from flask.ext.babel import gettext as _
```
使用方法是_("username")，也可以用lazy_gettext，不过实验中抛异常了，需要继续跟进

3 
编译cfgb
    把文件放到顶层目录中
```
pybabel extract -F babel.cfg -o messages.pot .
``` 
(这里.是目录)    
```
pybabel extract -F babel.cfg -k lazy_gettext -o messages.pot .
```
便根据gettext()的文字，自动生成pot文件

4 
由pot创建各国语言
```
pybabel init -i messages.pot -d translations -l de
```
```
pybabel init -d translations -l es -D flask_user -i translations/flask_user.pot
```
会生成 translations/de/LC_MESSAGES/messages.po 文件

5 修改po文件对应的字段
```
pybabel compile -d translations
```

```
pybabel compile -d translations -D flask_user -f
```

6 
如果字段有变化,修改pot文件，再
```
pybabel update -i messages.pot -d translations
```
```
pybabel update -d translations -l es -D flask_user -i translations/flask_user.pot
```
Use the browser's language preferences to select an available translation

```
    @babel.localeselectordef get_locale():
        translations = [str(translation) for translation in babel.list_translations()]
        return request.accept_languages.best_match(translations)
```



[1]:    {{ page.url}}  ({{ page.title }})
