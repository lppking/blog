根据官方文档，rn原生提供的组件中可以实现滚动列表的功能的组件包括`scrollView`、`FlatList`和`SectionList`。

## 1. ScrollView和FlatList
**ScrollView**的缺点很明显，就是会渲染所有的列表项，不管是可见的还是不可见的，这自然而然的就会带来一个明显的问题，当列表是一个数据项很多的长列表时，性能问题就变得非常突出了。
针对`ScrollView`的这个缺点，一种解决方法就是自己控制渲染的列表项的数量，这就需要自己通过JS来判断该显示哪些，什么时候添加新的，什么时候移除不展示的，如果列表项还是不规则的，比如高度是动态可变的，还需要考虑视图对象的创建和回收对性能的影响等等因素，这时候自己实现就会变得非常复杂。
另一种相对简单的选项就是使用官方提供的长列表组件**FlatList**。官方文档中给的推荐理由就是：并非渲染全部，优先渲染可见项。
使用`FlatList`组件，必须提供`data`和`renderItem`属性：
```
import React, { Component } from "react";

import { FlatList, StyleSheet, Text, View } from "react-native";

export default class FlatListDemo extends Component {



    renderItem = ({ item }) => {

        return <Text style={styles.item}>{item.key}</Text>;

    }



    render () {

        return (

            <View style={styles.container}>

                <FlatList

                    data={[

                        { key: "张三" },

                        { key: "李四" },

                        { key: "王五" },

                    ]}

                    renderItem={this.renderItem}

                />

            </View>

        )

    }

}

const styles = StyleSheet.create({

    container: {

        flex: 1,

        paddingTop: 22,

    },

    item: {

        padding: 10,

        fontSize: 18,

        height: 44,

    }

});

```
`FlatList`最大的优点就是长列表场景下的**高性能**，但是在实测中，在部分安卓老机型中也会存在明显的卡顿现象，**原因是快速滚动时需要大量删除和新增列表项对象，是需要消耗大量计算性能以及内存的， 所以在部分老安卓机型上存在不够流畅的问题。**组内大佬提供的说明文章可以[参考这里](https://segmentfault.com/a/1190000016959264)。

## 2. SectionList
如果要渲染的数据是分组数据，那么官方推荐使用`SectionList`组件。
```
import React, { Component } from "react";

import { SectionList, StyleSheet, Text, View } from "react-native";

export default class SectionListDemo extends Component {

  defaultValues = {

    sections: [

      {

        title: "水果",

        data: ["苹果", "香蕉", "菠萝"]

      },

      {

        title: "奶制品",

        data: ["酸奶", "纯奶", "核桃奶"]

      }

    ]

  };



  renderItem = ({ item }) => {

    return <Text style={styles.item}>{item}</Text>;

  };



  renderSectionHeader = ({ section }) => {

    return <Text style={styles.sectionHeader}>{section.title}</Text>;

  };



  render() {

    return (

      <View style={styles.container}>

        <SectionList

          sections={this.defaultValues.sections}

          renderItem={this.renderItem}

          renderSectionHeader={this.renderSectionHeader}

          keyExtractor={({ item, index}) => index}

        />

      </View>

    );

  }

}

const styles = StyleSheet.create({

  container: {

    flex: 1,

    paddingTop: 22

  },

  sectionHeader: {

    paddingTop: 2,

    paddingLeft: 10,

    paddingRight: 10,

    paddingBottom: 2,

    fontSize: 14,

    fontWeight: "bold",

    backgroundColor: "rgba(247,247,247,1.0)"

  },

  item: {

    padding: 10,

    fontSize: 18,

    height: 44

  }

});

```

## 3. recycler-list-view
这是一个第三方的rn滚动列表组件，作者对这个组件的简介如下：
> RecyclerListView uses "cell recycling" to reuse views that are no longer visible to render items instead of creating new view objects. Creation of objects is very expensive and comes with a memory overhead which means as you scroll through the list the memory footprint keeps going up. Releasing invisible items off memory is another technique but that leads to creation of even more objects and lot of garbage collections. Recycling is the best way to render infinite lists that does not compromise performance or memory efficiency.

大意就是他们使用了一种被称作**cell recycling**单元格回收的概念来替代传统的**销毁不可见项对象，新增可见项创建新视图对象的方式。**因为创建对象是一个非常消耗性能的操作，这就意味着你在快速滚动列表的时候内存消耗会大量增加。回收不可见元素的视图对象在作者看来是一个浪费的行为，不断的销毁导致必须不断的创建，所以从简介大致能了解到作者是通过复用已有的视图对象来达到减少新建和销毁操作来提高性能的。有兴趣的同学可以[点击这里](https://github.com/Flipkart/recyclerlistview)查看详细内容。
使用`recyclerlistview`组件的核心是理解`dataProvider`、`layoutProvider`以及`rowRenderer`这三个必要属性。
## 3.1 dataProvider
dataProvider包含两个部分：
首先是new一个dataProvider实例，提供一个参数用于比较两条数据是否为同一条数据，即是否为同一个对象。
```
/**

 * dataProvider是必须提供的，用于比较两行数据是否是同一个

 */

let dataProvider = new DataProvider((r1, r2) => r1 !== r2);

```
然后是使用DataProvider实例的`cloneWithRows`方法，传入数据数组作为参数，返回经过处理包装后的一个`DataProvider`实例对象作为`this.state.dataProvider`的值，最后将这个值传递给`RecyclerListView`的`dataProvider`属性。
```
this.state = {

       dataProvider: dataProvider.cloneWithRows(this._generateArray(100)), // 生产一个包含100个元素的数组

}

```
这部分是对数据做处理。

## 3.2 layoutProvider
这部分是对视图元素做处理。首先提供一个视图元素类型对象，方便在后续对视图元素分类做映射：
```
/**
 * 仅作为一个类型标示，key和value都可以自定义
 */
const ViewTypes = {
    FULL: 0,
    HALF_LEFT: 1,
    HALF_RIGHT: 2,
};
```
然后new一个`LayoutProvider`实例，提供两个参数，参数含义见注释：
```
        /**
         * layoutProvider也是必须提供的
         * param1: 第一个参数是根据索引返回项目类型，这个类型对应viewTypes中预设的类型，
         *          这个函数作用是给第二个参数根据type对相应的列表项做差异化处理准备的
         * param2: 第二个参数是根据type设置布局
         * 
         * 官方说明中提了一嘴说如果需要根据数据做筛选，可以在这里访问dataProvider
         */
        this._layoutProvider = new LayoutProvider(
            index => {
                if (index % 3 === 0) {
                    return ViewTypes.FULL;
                } else if (index % 3 === 1) {
                    return ViewTypes.HALF_LEFT;
                } else {
                    return ViewTypes.HALF_RIGHT;
                }
            },
            (type, dim) => {
                switch (type) {
                    case ViewTypes.FULL:
                        dim.width = width;
                        dim.height = 140;
                        break;
                    case ViewTypes.HALF_LEFT:
                        dim.width = width / 2;
                        dim.height = 160;
                        break;
                    case ViewTypes.HALF_RIGHT:
                        dim.width = width / 2;
                        dim.height = 160;
                        break;
                    default: 
                        dim.width = 0;
                        dim.height = 0;
                }
            }
        )
```
可以看出来，第一个参数是根据`type`返回数据元素对应的`ViewTypes`类型，第二个参数是根据元素的`ViewTypes`类型对视图元素做处理。

## 3.3 rowRenderer
这个东西很好理解，首先它的作用是返回列表项的视图元素，具体返回什么样的视图，可以自定义判断条件。
```
/**
     * 提供type data返回视图组件
     * 这里可以按需根据type返回不同视图元素
     */
    _rowRenderer (type, data) {
        // 在这里可以返回任意视图，CellContainer这个类在这里没有什么特殊意义
        switch (type) {
            case ViewTypes.FULL:
                return (
                    <CellContainer style={styles.container}>
                        <Text>Data: {data}</Text>
                    </CellContainer>
                );
            case ViewTypes.HALF_LEFT:
                return (
                    <CellContainer style={styles.containerGridLeft}>
                        <Text>Data: {data}</Text>
                    </CellContainer>
                );
            case ViewTypes.HALF_RIGHT: 
                return (
                    <CellContainer style={styles.containerGridRight}>
                        <Text>Data: {data}</Text>
                    </CellContainer>
                );
            default: 
                return null;
        }
    }
```
这里用的`CellContainer`没有特殊含义，完全是自定义的。

## 4. 一个简单的recycler-list-view使用示例
使用实例参照官方demo，增加了一些注释帮助理解，个人感觉还是照着例子写一遍，能更快速的掌握基本用法：
```
import React, { Component } from 'react';

import { View, Text, Dimensions } from "react-native";

import { RecyclerListView, DataProvider, LayoutProvider } from "recyclerlistview";



/**

 * 仅作为一个类型标示，key和value都可以自定义

 */

const ViewTypes = {

    FULL: 0,

    HALF_LEFT: 1,

    HALF_RIGHT: 2,

};



let containerCount = 0;



/**

 * 单元格容器

 */

class CellContainer extends Component {

    constructor (props) {

        super(props);

        this._containerId = containerCount++;

    }



    render () {

        return (

            <View

                {...this.props}

            >

                { this.props.children }

                <Text>单元格ID: { this._containerId }</Text>

            </View>

        )

    }

}



export default class RecycleTestComponent extends Component {

    constructor (props) {

        super(props);



        let { width } = Dimensions.get("window");



        /**

         * dataProvider是必须提供的，用于比较两行数据是否是同一个

         */

        let dataProvider = new DataProvider((r1, r2) => r1 !== r2);



        /**

         * layoutProvider也是必须提供的

         * param1: 第一个参数是根据索引返回项目类型，这个类型对应viewTypes中预设的类型，

         *          这个函数作用是给第二个参数根据type对相应的列表项做差异化处理准备的

         * param2: 第二个参数是根据type设置布局

         * 

         * 官方说明中提了一嘴说如果需要根据数据做筛选，可以在这里访问dataProvider

         */

        this._layoutProvider = new LayoutProvider(

            index => {

                if (index % 3 === 0) {

                    return ViewTypes.FULL;

                } else if (index % 3 === 1) {

                    return ViewTypes.HALF_LEFT;

                } else {

                    return ViewTypes.HALF_RIGHT;

                }

            },

            (type, dim) => {

                switch (type) {

                    case ViewTypes.FULL:

                        dim.width = width;

                        dim.height = 140;

                        break;

                    case ViewTypes.HALF_LEFT:

                        dim.width = width / 2;

                        dim.height = 160;

                        break;

                    case ViewTypes.HALF_RIGHT:

                        dim.width = width / 2;

                        dim.height = 160;

                        break;

                    default: 

                        dim.width = 0;

                        dim.height = 0;

                }

            }

        )



        this._rowRenderer = this._rowRenderer.bind(this);



        this.state = {

            dataProvider: dataProvider.cloneWithRows(this._generateArray(100)), // 生产一个包含100个元素的数组

        }

    }



    /**

     * 提供type data返回视图组件

     * 这里可以按需根据type返回不同视图元素

     */

    _rowRenderer (type, data) {

        // 在这里可以返回任意视图，CellContainer这个类在这里没有什么特殊意义

        switch (type) {

            case ViewTypes.FULL:

                return (

                    <CellContainer style={styles.container}>

                        <Text>Data: {data}</Text>

                    </CellContainer>

                );

            case ViewTypes.HALF_LEFT:

                return (

                    <CellContainer style={styles.containerGridLeft}>

                        <Text>Data: {data}</Text>

                    </CellContainer>

                );

            case ViewTypes.HALF_RIGHT: 

                return (

                    <CellContainer style={styles.containerGridRight}>

                        <Text>Data: {data}</Text>

                    </CellContainer>

                );

            default: 

                return null;

        }

    }



    _generateArray (n) {

        return new Array(n).fill(1).map((item, index) => index);

    }



    render () {

        return (

            <RecyclerListView

                layoutProvider={this._layoutProvider}

                dataProvider={this.state.dataProvider}

                rowRenderer={this._rowRenderer}

            ></RecyclerListView>

        )

    }

}



const styles = {

    container: {

        justifyContent: "space-around",

        alignItems: "center",

        flex: 1,

        backgroundColor: "#00a1f1"

    },

    containerGridLeft: {

        justifyContent: "space-around",

        alignItems: "center",

        flex: 1,

        backgroundColor: "#ffbb00"

    },

    containerGridRight: {

        justifyContent: "space-around",

        alignItems: "center",

        flex: 1,

        backgroundColor: "#7cbb00"

    }

};

```
**以上就是我所了解的在rn中实现滚动列表的几种方式，如果还有更好的实现方式，欢迎在评论区留言。**
