# 瓦片底图制作与发布教程
©2017 NavData. All rights reserved.
+ 本教程公开，允许转载，但必须保留版权说明
+ 正式使用时请通过合法渠道获取底图数据或服务
+ 请注意互联网地图相关法律、法规及规定

# Todo list

    x: 未动工
    -: 已开始
    o: 已完成

 + 瓦片地图相关基础知识  (-)
 + 准备工作 (x)
 + 栅格瓦片制作 (-)
 + 矢量瓦片制作 (-)
 + 性能优化 (-)

# 1. 瓦片地图相关基础知识

## 1.3 瓦片地图的加载

### 1.3.1 直接根据图片URL加载

只要能够解析URL规则，并且与谷歌地图的x、y、z编号对应起来，直接使用图片URL可能是最简单直接的瓦片地图使用方式。请参见
http://openlayers.org/en/latest/apidoc/ol.source.XYZ.html
。

`注意：很多在线地图不允许（或者未公开声明允许）使用这种方式进行访问！`


### 1.3.2 使用WMTS


# 2. 准备工作

# 3. 栅格瓦片制作

## 3.1 ArcGIS路线

## 3.2 GeoServer路线

### 3.2.1 工具列表

+ Udig

    Udig是在Eclipse的基础上开发出来的，习惯Eclipse的用着会比较习惯。建议使用64位版，否则打开大量图层会比较痛苦，甚至出错

+ GeoServer


+ Tomcat

    存在更高效率的方式，暂不分析

### 3.2.2 样式设置

1. Udig新建项目`TileTest`

2. 项目中新建Map

3. Map添加图层，此处用广东地市边界

4. 设定图层的Style，包括颜色、填充、标签等，保存相应的XML数据，例如
```XML
<?xml version="1.0" encoding="UTF-8"?><sld:StyledLayerDescriptor xmlns="http://www.opengis.net/sld" xmlns:sld="http://www.opengis.net/sld" xmlns:gml="http://www.opengis.net/gml" xmlns:ogc="http://www.opengis.net/ogc" version="1.0.0">
    <sld:UserLayer>
        <sld:LayerFeatureConstraints>
            <sld:FeatureTypeConstraint/>
        </sld:LayerFeatureConstraints>
        <sld:UserStyle>
            <sld:Name>地级市界</sld:Name>
            <sld:FeatureTypeStyle>
                <sld:Name>group 0</sld:Name>
                <sld:FeatureTypeName>Feature</sld:FeatureTypeName>
                <sld:SemanticTypeIdentifier>generic:geometry</sld:SemanticTypeIdentifier>
                <sld:SemanticTypeIdentifier>simple</sld:SemanticTypeIdentifier>
                <sld:Rule>
                    <sld:Name>default rule</sld:Name>
                    <sld:PolygonSymbolizer>
                        <sld:Fill>
                            <sld:CssParameter name="fill">#C0C0C0</sld:CssParameter>
                            <sld:CssParameter name="fill-opacity">0.5</sld:CssParameter>
                        </sld:Fill>
                        <sld:Stroke/>
                    </sld:PolygonSymbolizer>
                    <sld:TextSymbolizer>
                        <sld:Label>
                            <ogc:PropertyName>NAME</ogc:PropertyName>
                        </sld:Label>
                        <sld:Font>
                            <sld:CssParameter name="font-family">Microsoft YaHei UI</sld:CssParameter>
                            <sld:CssParameter name="font-size">16.0</sld:CssParameter>
                            <sld:CssParameter name="font-style">normal</sld:CssParameter>
                            <sld:CssParameter name="font-weight">bold</sld:CssParameter>
                        </sld:Font>
                        <sld:LabelPlacement>
                            <sld:PointPlacement>
                                <sld:AnchorPoint>
                                    <sld:AnchorPointX>0.0</sld:AnchorPointX>
                                    <sld:AnchorPointY>0.0</sld:AnchorPointY>
                                </sld:AnchorPoint>
                                <sld:Displacement>
                                    <sld:DisplacementX>0.0</sld:DisplacementX>
                                    <sld:DisplacementY>0.0</sld:DisplacementY>
                                </sld:Displacement>
                            </sld:PointPlacement>
                        </sld:LabelPlacement>
                        <sld:Halo>
                            <sld:Radius>1</sld:Radius>
                            <sld:Fill>
                                <sld:CssParameter name="fill">#FFFFFF</sld:CssParameter>
                            </sld:Fill>
                        </sld:Halo>
                        <sld:Fill>
                            <sld:CssParameter name="fill">#000040</sld:CssParameter>
                        </sld:Fill>
                    </sld:TextSymbolizer>
                </sld:Rule>
            </sld:FeatureTypeStyle>
        </sld:UserStyle>
    </sld:UserLayer>
</sld:StyledLayerDescriptor>
```

5. 在GeoServer中发布相应的数据，设置样式

6. 制作瓦片cache

编号规则探索

结论：

    google x = tile x
    google z = tile z
    google y = tile

 `2^( 1 +  ( z / 2 ))`


+ 1级 2^0+1

    google    x   y   z 1   0   1

    tile  x y z  1  1 1-0_0

    offset   0  +1  0  

+ 2级 2^0+1

    google    x   y   z 3   1   2

    tile  x y z  3  2  2-0_0

    offset   0  +1  0  

+ 3级 2^0+1

    google  x  y  z   6  3  3

    tile  x  y  z    6  4  3-1_1

    offet 0 +1 0

+ 4级  2^0+1

    google  x  y  z   12 7 4

    tile  x  y  z     12  8  4-1_1

    offset 0 +1 0

+ 5级  2^1+1

    google  x  y  z   25 14 5

    tile  x  y  z     25  17  5-3_2

    offset 0 +3 0    

+ 6级 2^3+1

    google  x  y  z   51 28 6

    tile  x  y  z     51  35  6-3_2

    offset 0 +7 0


+ 7级 2^4+1

    google x y z   103 56 7

    tile x  y  z   103 71 7-06_04

    offset 0 +17 0

+ 8级 2^5+1

    google x y z  208  111  8

    tile x y z 208 144 08-06_04

    offset 0 +33 0

+ 9级  2^6+1

    google x y z   417   223    9

    tile  x y z   417  288   9-13_09

    offset 0 +65 0
为了高亮相对正常先按c语言标记了

```c
//初步转换成功
int z=9;
int x=417;
int y=223;
xyz_convert= convert(z,x,y);
x=xyz_convert[0];
y=xyz_convert[1];
z=xyz_convert[2];
int shift = z / 2;
half = 2 << shift;
int digits = 1;
if (half > 10){
	digits =(int) (Math.log(half)/Math.log(10)) + 1;
}
int halfx = x / half;
int halfy = y / half;

println (x);
println (y);
println (z);
println (halfx);
println (halfy);

 convert(z, x, y) {
	extent = Math.pow(2, z)
	if (x < 0 || x > extent - 1) {
		println("The X coordinate is not sane: " + x)
		return
	}
	if (y < 0 || y > extent - 1) {
		println("The Y coordinate is not sane: " + y)
		return
	}
	gridLoc = [x, extent - y - 1, z]
	return gridLoc
}



```




# 4. 矢量瓦片制作

### 4.1 工具列表

+ Udig

  现有版本疑似不支持多线程，不过可以手工分级并行

+ Tomcat


# 5. 性能优化

## 5.1 并行化渲染及切图

## 5.2 缓存优化

## 5.3 瓦片服务负载均衡

目前很多WebGIS类库允许在瓦片地图URL上使用服务器通配符，例如国内某地图服务有t0.\*\*\*\*\*\*\*\*.cn到t7.\*\*\*\*\*\*\*\*\.cn共8台服务器，其访问地址可写为：

    http://t{0-7}.******.cn/DataServer?T=vec_w&X={x}&Y={y}&L={z}

这样浏览器访问该服务时会轮换使用几个服务器，在一定程度上实现了负载均衡。采用这种方式的话，通常不必主动进行负载均衡处理。
