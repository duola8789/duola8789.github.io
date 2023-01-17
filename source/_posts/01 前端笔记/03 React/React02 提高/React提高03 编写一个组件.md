---
title: React提高03 编写一个组件
top: false
date: 2019-01-25 16:04:43
updated: 2019-01-25 16:04:43
tags:
- React
- 坑
categories: React
---

以前总结的，简单的编写一个组件的过程

还是太简略了，以后慢慢丰富吧。

<!-- more -->

## 详解过程

编写一个组件首先引入相关模块
- `React`
- `prop-types`

然后声明一个继承自`React.Component`的类：

```
export default class MyComponent extends from Reac.Component{
    
}
```
然后对传入的`Prop`进行限定和验证：

```
static PropTypes = {
  checkedText: PropTypes.string,
  uncheckedText: PropTypes.string,
  hideText: PropTypes.bool,
  onClickFunc: PropTypes.func
}
```
可以对`Prop`的默认值进行预定义：


```
static defaultProp = {
  checkedText: '开',
  uncheckedText: '关',
  hideText: false
}
```
然后定义组件的`constructor`， 在其中可以定义`state`中的初始值

```
constructor(props) {
  super(props);
  this.state = {
    checked: false
  }
};
```
将数组传回父组件，是通过调用父组件的方法，父组件的方法是通过Prop传入子组件中进行调用的

```
// 父组件中引用子组件
<MyInput onClickFunc={this.handleClick.bind(this)}/>

// 子组件中引用父组件的方法并传值
handleClick() {
  const {checked} = this.state;
  const {onClickFunc} = this.props;
  this.setState({
    checked: !checked
  }, () = > {
    onClickFunc(this.state.checked)
  });
```
注意，调用父组件的方法放在了`setState`方法的回调函数中

## 一个组件的例子

定义子组件
```
/**
 * Created by zh on 2018/1/24.
 */
import React from 'react';
import PropTypes from 'prop-types';
import ReactDOM from 'react-dom';
import styles from './myInput.css';
import classnames from 'classnames'

export default class MyInput extends React.Component {

  static propTypes = {
    checkedText: PropTypes.string,
    uncheckedText: PropTypes.string,
    hideText: PropTypes.bool,
    onClickFunc: PropTypes.func
  };

  static defaultProps = {
    checkedText: '开',
    uncheckedText: '关',
    hideText: false
  };

  constructor(props) {
    super(props);
    this.state = {
      checked: false
    }
  };


  handleClick() {
    const {checked} = this.state;
    const {onClickFunc} = this.props;

    this.setState({
      checked: !checked
    }, () =>{
      onClickFunc(this.state.checked)
    });
  }

  render() {
    const {checkedText, uncheckedText, hideText} = this.props;
    const {checked} = this.state;

    let checkStyle = checked ? styles['circleChecked'] : styles['circleUnchecked'];

    let textEle = !hideText ?
      (
        <span className={ checked ? styles.checkedText : styles.unCheckedText}>
          {checked ? checkedText : uncheckedText}
        </span>
      )
      : null;

    return (
      <div className={styles.container} onClick={this.handleClick.bind(this)}>
        <div className={classnames(styles['circle'], checkStyle)}/>
        {textEle}
      </div>
    )
  }
}
```
引用子组件

```
/**
 * Created by zhouhao on 2017/5/8.
 */
import React from 'react';
import ReactDOM from 'react-dom';
import {observable, computed, action} from 'mobx';
import {observer} from 'mobx-react'
import Store from './components/Store';
import style from '../css/root.css';
import MyInput from './components/MyInput';

@observer
export default class Root extends React.Component {

  constructor() {
    super();
    this.state = {
      childState: '关'
    }
  }

  handleClick(inputChecked){
    let state = inputChecked ? '开' : '关';
    this.setState({
      childState: state
    });
  };

  render() {
    let {childState} = this.state;
    return (
      <div>
        <MyInput onClickFunc={this.handleClick.bind(this)}/>
        <p>子组件状态:{childState}</p>
      </div>
    )
  }
}
```
