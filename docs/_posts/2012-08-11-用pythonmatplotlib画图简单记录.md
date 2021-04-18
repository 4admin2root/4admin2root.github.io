---
layout:     post
title:      用pythonmatplotlib画图简单记录
date:       2012-08-11
---
我们团队准备改造现有的监控机制由于性能原因考虑将业务数据趋势图由VBA生成改造成用python的matplotlib顺便记录下一个cookbookhttp://www.scipy.org/Cookbook-------还有http://www.lfd.uci.edu/~gohlke/pythonlibs/===========================================依次安装python 2.7numpy-MKL-1.6.2.win32-py2.7matplotlib-1.1.0.win32-py2.7basemap-1.0.4.win32-py2.7（测试下）----------------------

（办公室网连接sourceforge 好像不行了）

本人山寨代码如下

from mpl_toolkits.basemap import Basemapimport matplotlib.pyplot as pltimport numpy as np# set up orthographic map projection with# perspective of satellite looking down at 50N, 100W.# use low resolution coastlines.# don't plot features that are smaller than 1000 square km.map = Basemap(projection='ortho', lat_0 = 30, lon_0 = 100,              resolution = 'l', area_thresh = 1000.)# draw coastlines, country boundaries, fill continents.map.drawcoastlines()map.drawcountries()map.fillcontinents(color = 'coral')# draw the edge of the map projection region (the projection limb)map.drawmapboundary()# draw lat/lon grid lines every 30 degrees.map.drawmeridians(np.arange(0, 360, 30))map.drawparallels(np.arange(-90, 90, 30))#lat/lon coordinates of five cities.lats = [39.9, 22.5]lons = [116.4, 114]cities=['BeiJing','ShenZhen']# compute the native map projection coordinates for cities.x,y = map(lons,lats)# plot filled circles at the locations of the cities.map.plot(x,y,'bo')# plot the names of those five cities.for name,xpt,ypt in zip(cities,x,y):    plt.text(xpt+50000,ypt+50000,name)plt.show()

-------------

原代码参见[http://www.scipy.org/Cookbook/Matplotlib/Maps](http://www.scipy.org/Cookbook/Matplotlib/Maps)

效果如下

<img src="http://my.csdn.net/uploads/201208/03/1343975123_1938.png" alt="" />
