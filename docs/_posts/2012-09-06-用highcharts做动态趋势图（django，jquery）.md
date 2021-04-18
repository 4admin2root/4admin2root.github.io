---
layout:     post
title:      用highcharts做动态趋势图（django，jquery）
date:       2012-09-06
tags: [cnblogs]
---
用highcharts做动态趋势图 （django，jquery）

highcharts官方有个动态图的demo（Spline updating each second）[http://www.highcharts.com/demo/dynamic-update](http://www.highcharts.com/demo/dynamic-update)

觉得效果不错，作为维护工作用的监控很合适

于是进行丰富（以下代码仅作参考，未考虑异常和安全）==============================1、后台返回y值

使用django框架

from  mainapp.models import mymodel 

def dynamic_update(request): dy_data = mymodel.objects.filter(xxx='111') dy_id = dy_data.aggregate(Max('id')) dy_max_data = mymodel.objects.filter(id=dy_id['id__max']) html = '' json_serializer = serializers.get_serializer("json")() html += json_serializer.serialize(dy_max_data) return HttpResponse(html,mimetype="text/json")

可以先用浏览器测试下

2、在动态图demo的脚本中增加一个函数

 function getAjaxvalue()    {          var y = 0;   $.ajax({      type:"GET",      async:false,  [url:"/dynamic_update/](http://www.cnblogs.com/4admin2root/admin/%22/dynamic_update/)",  datatype:"json",      success:function(result){  $.each(result,function(idx,item){                     y = item.fields['value']                            });  }    });  return y; }

打开浏览器看看吧

后面还需要修改间隔时间 还有 就是 初始化的20条 随机数 我简单设置为0了
