# mip-map 地图组件

`<mip-map>` 组件集成了百度地图的服务，目前支持定位、地图控件加载、定位点弹窗信息定制等功能！

| 标题     | 内容                                              |
| -------- | ------------------------------------------------- |
| 类型     | 通用                                              |
| 支持布局 | responsive, fixed-height, fill, container, fixed  |
| 所需脚本 | https://c.mipcdn.com/static/v2/mip-map/mip-map.js |

## 示例

### 基本用法

```html
<mip-map height="400">
  <script type="application/json">
    {
      "ak": "hKhuzfFBrcL6zGm4s6b371NDxaUrhFPl",
      "location": {
        "province": "北京",
        "city": "北京市",
        "district": "海淀区",
        "street": "百度大厦"
      },
      "controls": {
        "NavigationControl": {
        },
        "MapTypeControl": {
        }
      },
      "info": {
        "width" : 250,
        "height": 100,
        "content": "<h4>百度大厦</h4><p>地址：北京市海淀区上地十街10号<br/>电话：(010)59928888<br/>简介：百度大厦位于北京市海淀区西二旗地铁站附近，为百度公司综合研发及办公总部。</p></div>"
      }
    }
  </script>
</mip-map>
```

## 使用说明

地图组件具体参数的配置和使用方式如下：

### ak

使用地图组件之前必须要申请成为百度开发者，并创建百度服务密钥（`ak`），这里的 `ak` 参数即代表该功能。具体申请方式可以参见[百度地图 Javascript api 文档](http://lbsyun.baidu.com/index.php?title=jspopular/guide/getkey)。

### location

定位相关参数，可以根据配置的具体地址来获取在地图上的精确定位。

- `province`: 定位所对应的省份或直辖市，如“北京”。
- `city`: 定位所对应的城市，如“北京市”。
- `district`: 定位所对应的区或县，如“海淀区”。
- `street`: 定位所对应的具体街道或门牌信息，如“西北旺东路 10 号院”。

### controls

在地图绘制完成之后也可以在地图中加入对应的控件，具体是按照 `控件名: 控件参数对象` 的方式设置，如：

```
<script type="application/json">
  {
    "ak": "hKhuzfFBrcL6zGm4s6b371NDxaUrhFPl",
    "location": {
      "province": "北京",
      "city": "北京市",
      "district": "海淀区",
      "street": "百度大厦"
    },
    "controls": {
      "NavigationControl": {
      },
      "MapTypeControl": {
      }
    },
    "info": {
      "width" : 250,
      "height": 100,
      "content": "<h4>百度大厦</h4><p>地址：北京市海淀区上地十街10号<br/>电话：(010)59928888<br/>简介：百度大厦位于北京市海淀区西二旗地铁站附近，为百度公司综合研发及办公总部。</p></div>"
    }
  }
</script>
```

具体控件列表和参数可以参考百度地图提供的[控件文档](http://lbsyun.baidu.com/cms/jsapi/reference/jsapi_reference.html#a2b0)。

### info

地图定位后，点击定位点会出现具体弹层，弹层的各个参数都是可配置的，弹层内容可以通过 `content` 参数来设置，`content` 的值为 HTML 类型，其他参数可参考百度地图提供的[弹层文档](http://lbsyun.baidu.com/cms/jsapi/reference/jsapi_reference.html#a3b8)。
