## setState

### 为什么setState不同步更新this.state？
参考：https://zhuanlan.zhihu.com/p/25990883

为什么React不能立即更新this.state，而不对组件进行重新渲染？

https://github.com/facebook/react/issues/11527#issuecomment-360199710

有这样一种情况，更新state后要访问state，虽然setState提供callback，但是有时访问这个新state的代码，并不能放到callback里。这时可以直接用this.state设置值，然后调用setState提示react触发渲染。不过官方建议不要把这些要即时访问和修改的状态放到state里。

```js
import React from 'react';

export default class Test extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      hasClicked: false,
    };
  }

  handleClick = () => {
    console.log(this.state.hasClicked); // 极速地点击下面按钮两次，这里有可能打印两次false吗？实际测试过，是不可能出现旧state的。所以应该可以相信，连续触发两个event，后来的handler里取到的state是新的。
    this.setState({
      hasClicked: true
    });
  }

  render() {
    return (<button onClick={this.handleClick}>test</button>);
  }
}
```