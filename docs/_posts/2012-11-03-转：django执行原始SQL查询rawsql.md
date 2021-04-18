---
layout:     post
title:      转：django执行原始SQL查询rawsql
date:       2012-11-03
---
摘自The Django Book第10章： 数据模型高级进阶[http://djangobook.py3k.cn/2.0/chapter10/](http://djangobook.py3k.cn/2.0/chapter10/)

[https://docs.djangoproject.com/en/dev/topics/db/sql/](https://docs.djangoproject.com/en/dev/topics/db/sql/)

django 执行原始SQL查询有时候你会发现Django数据库API带给你的也只有这么多，那你可以为你的数据库写一些自定义SQL查询。 你可以通过导入django.db.connection对像来轻松实现，它代表当前数据库连接。 要使用它，需要通过connection.cursor()得到一个游标对像。 然后，使用cursor.execute(sql, [params])来执行SQL语句，使用cursor.fetchone()或者cursor.fetchall()来返回记录集。 例如:>>> from django.db import connection>>> cursor = connection.cursor()>>> cursor.execute("""...    SELECT DISTINCT first_name...    FROM people_person...    WHERE last_name = %s""", ['Lennon'])>>> row = cursor.fetchone()>>> print row['John']connection和cursor几乎实现了标准Python DB-API，你可以访问` http://www.python.org/peps/pep-0249.html <http://www.python.org/peps/pep-0249.html>`__来获取更多信息。 如果你对Python DB-API不熟悉，请注意在cursor.execute() 的SQL语句中使用`` %s`` ，而不要在SQL内直接添加参数。 如果你使用这项技术，数据库基础库将会自动添加引号，同时在必要的情况下转意你的参数。不要把你的视图代码和django.db.connection语句混杂在一起，把它们放在自定义模型或者自定义manager方法中是个不错的主意。 比如，上面的例子可以被整合成一个自定义manager方法，就像这样：from django.db import connection, modelsclass PersonManager(models.Manager):    def first_names(self, last_name):        cursor = connection.cursor()        cursor.execute("""            SELECT DISTINCT first_name            FROM people_person            WHERE last_name = %s""", [last_name])        return [row[0] for row in cursor.fetchone()]class Person(models.Model):    first_name = models.CharField(max_length=50)    last_name = models.CharField(max_length=50)    objects = PersonManager()然后这样使用:>>> Person.objects.first_names('Lennon')['John', 'Cynthia']===================================================

[https://docs.djangoproject.com/en/dev/topics/db/sql/](https://docs.djangoproject.com/en/dev/topics/db/sql/)

# Performing raw SQL queries

When the [**model query APIs**](https://docs.djangoproject.com/en/dev/topics/db/queries/) dont go far enough, you can fall back to writing raw SQL. Django gives you two ways of performing raw SQL queries: you can use [<tt class="xref py py-meth docutils literal">Manager.raw()</tt>](https://docs.djangoproject.com/en/dev/topics/db/sql/#django.db.models.Manager.raw) to [perform raw queries and return model instances](https://docs.djangoproject.com/en/dev/topics/db/sql/#performing-raw-queries), or you can avoid the model layer entirely and [execute custom SQL directly](https://docs.djangoproject.com/en/dev/topics/db/sql/#executing-custom-sql-directly).
