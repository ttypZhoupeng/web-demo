<!--
 * @Description: 
 * @Autor: Zhoupeng
 * @Date: 2020-11-23 20:02:50
 * @LastEditors: Zhoupeng
 * @LastEditTime: 2020-11-24 10:08:15
-->
# 原生js实现框选表格内容并批量更改（vue）

## 前言 
日常开发中，有这样的需求，需要在表格中进行框选，并对框选中的内容进行批量更改。源码于文章结尾

## demo

![demo](https://github.com/ttypZhoupeng/web-demo/blob/main/table-region/1.png?raw=true)

下面链接供在线调试：

https://codesandbox.io/s/table-region-wxky2?file=/index.html


## 讲解
主要思路通过监听鼠标事件，进行框选框绘制。获取框选 的dom，并计算其是否在框选区域，来进行批量更改。其中主要的方式 [getClientRects](https://developer.mozilla.org/zh-CN/docs/Web/API/Range/getClientRects)。

#### 业务代码：

**1、初始化表格数据结构**

因为表格为星期/时间，相对结构会有点子复杂，问题不大，giao。
```javascript
setTableConfig() {
    let timeThArr = Array.from(new Array(24).keys());
    let timeTdArr = Array.from(new Array(48).keys());
    timeTdArr = timeTdArr.map((item) => {
      let obj = {
        id: item,
      };
      return obj;
    });
    this.timeThArr = timeThArr;
    this.timeTdArr = timeTdArr;

    let tableList = this.tableData;
    tableList.map((item, index) => {
      this.timeThArr.map((temp, key) => {
        item.push({
          key: key + index * 24,
        })
      })
    })
    this.tableData = tableList;
  },
```

**2、点击单格选中**

除了框选也可以单击选中
```javascript
handleClickTimeData(obj, index) {
    let tableList = _.clone(this.tableData);
    _.map(tableList[index], (item) => {
      if (item.key == obj.key) {
        item.checked = !item.checked;
      }
    });
    this.tableData = tableList;
},   
```

**3、重置**

取消所有框选
```javascript
reset() {
    let tableList = _.clone(this.tableData);
    _.map(tableList, (item) => {
      _.map(item, temp => {
        temp.checked = false;
      })
    });
    this.tableData = tableList;
},
```

#### 框选代码

**1、表格绑定mousedown、mousemove、mouseup事件**

```
handleMouseDown(e) {
    this.is_show_mask = true;
    this.start_x = e.clientX;
    this.start_y = e.clientY;
    this.end_x = e.clientX;
    this.end_y = e.clientY;
    document.body.addEventListener("mousemove", this.handleMouseMove);
    document.body.addEventListener("mouseup", this.handleMouseUp);
},
handleMouseMove(e) {
    this.end_x = e.clientX;
    this.end_y = e.clientY;
},
handleMouseUp() {
    document.body.removeEventListener("mousemove", this.handleMouseMove);
    document.body.removeEventListener("mouseup", this.handleMouseUp);
    this.is_show_mask = false;
    this.handleDomSelect();
    this.resetXY();
},
```

**2、计算dom是否在包含在框选区域内部**

```
collide(rect1, rect2) {
    const maxX = Math.max(rect1.x + rect1.width, rect2.x + rect2.width);
    const maxY = Math.max(rect1.y + rect1.height, rect2.y + rect2.height);
    const minX = Math.min(rect1.x, rect2.x);
    const minY = Math.min(rect1.y, rect2.y);
    if (
      maxX - minX <= rect1.width + rect2.width &&
      maxY - minY <= rect1.height + rect2.height
    ) {
      return true;
    } else {
      return false;
    }
},
```

**3、鼠标松开后进行一系列操作**

（1）首先获取框选的dom，主要的方法为[getClientRects](https://developer.mozilla.org/zh-CN/docs/Web/API/Range/getClientRects)
（2）然后计算获取的元素是否在框选范围内
（3）对范围内的dom进行后续操作

```
handleDomSelect() {
    const dom_mask = window.document.querySelector(".mask");
    const rect_select = dom_mask.getClientRects()[0];
    
    let selectKeys = [];
    document.querySelectorAll(".week-data-td").forEach((node, index) => {
      const rects = node.getClientRects()[0];
      if (this.collide(rects, rect_select) === true) {
        selectKeys.push(index);
      }
    });
    if (selectKeys.length < 2) return;
    let tableList = _.clone(this.tableData);
    tableList = _.map(tableList, (item, key) => {
      return _.map(item, (temp) => {
        if (selectKeys.indexOf(temp.key) > -1) {
          temp.checked = true;
        }
        return temp;
      });
    });
    this.tableData = tableList;
},
```

框选这部分代码，具体参考大佬代码 

http://www.360doc.com/content/20/0702/14/26535044_921856251.shtml

## 源码

https://github.com/ttypZhoupeng/web-demo/tree/main/table-region

## 末了

如果这篇文章对大佬有帮助，请不要吝啬你的赞。来个一键三连就更好了，感谢大佬参阅。