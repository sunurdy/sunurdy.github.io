---
layout:     post
title:     Task工作流引擎
category: blog
description: 工作
---

##Task工作流引擎项目总结

最近做的预研项目，离线批处理任务引擎。类似stackstorm等引擎，最早是看到某公司做的playbook，想自己开发一版。

旁杂的暂且不表，一直是推翻重写，意见总是不一致。不过每一次都会使用到python中的以前不常用的小魔术，记一下。

# 动态生成类

这个貌似最常用了，和wtform组合起来验证用户输入，非常方便。
···
	required_fields = {
        'index': StringField(label=u'索引类型', description=u'请输入索引类型',
                             validators=[Length(min=0, max=40, message=u'索引类型长度应是1至40位')]),

        'start': IntegerField(label=u'起始条数', description=u'请输入0到1000000的起始条数', default=0,
                              validators=[NumberRange(min=0, max=10, message=u'起始条数应处于0至1000000范围内')]),
        'size': IntegerField(label=u'返回条数', description=u'请输入0到1000的返回条数',
                             validators=[DataRequired(message=u'no size'),
                                         NumberRange(min=0, max=1000, message=u'起始条数应处于0至1000范围内')])
    }
	form_class = type("tmp", (wtforms.Form,), required_fields)
	form = form_class(formdata=MultiDict(kwargs))
	if not form.validate():
		raise Exception(form.errors)
	return form
···

这个魔法在orm中很常用，比如sqlalchemy，具体是什么来着，我还得再看下，又忘了

# 提取被装饰的函数

# 从配置文件生成类

# 
