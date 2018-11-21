---
title: 【译】React v16.4.0：你可能并不需要派生状态（Derived State）
date: 2018-06-27 18:44:43
tags:
---
   
![](https://user-gold-cdn.xitu.io/2018/6/27/16440e1f61982561?w=720&h=400&f=png&s=50702)
 
 很长一段时间，`componentWillReceiveProps`生命周期是在不进行额外render的前提下，响应props中的改变并更新state的唯一方式。在16.3版本中，[我们介绍了一个新的替代生命周期getDerivedStateFromProps](https://reactjs.org/blog/2018/03/29/react-v-16-3.html#component-lifecycle-changes)去更安全地解决相同的问题。同时，我们意识到人们对这两个方式都存在很多的误解，我们发现了其中的一些反例触发了一些奇怪的bug。在本次版本中我们修复了它，[并让derived state更加可预测](https://github.com/facebook/react/issues/12898)，所以我们能更容易地注意到滥用的结果。
 
 > 本文的反例既包含老的componentWillReceiveProps也包含新的getDerivedStateFromProps方法
 
 ## 什么时候去使用派生状态(Derived State)
 
 `getDerivedStateFromProps`只为了一个目的存在。它使得一个组件能够响应props的变化来更新自己内部的state。比如我们之前提到的[根据变化的offset属性记录目前的滚动方向](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html#updating-state-based-on-props)或者[根据source属性加载额外的数据](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html#fetching-external-data-when-props-change)。
 
 我们提供了许多了实例，因为一般来说，**派生状态应该被谨慎地使用**。我们见过的所有关于派生状态的问题最后都可以被归为两种：（1）从props那里无条件地更新state（2）当props和state不匹配的时候更新state（我们在下面会深入探讨）
 
 - 如果你使用派生状态来记忆基于当前props的运算结果，你并不需要它。参见下面的`关于缓存记忆（memoization）`。
 - 如果你是第二种情况，那么你的组件可能重置得太频繁了。继续读下去获得更多内容。
 
## 使用派生状态的常见Bug

["受控"](https://reactjs.org/docs/forms.html#controlled-components)和["非受控"](https://reactjs.org/docs/uncontrolled-components.html)通常指代表单的输入控件，但是它还可以用于描述组件的数据所处位置。通过props传入的数据可被称为**受控的**（因为父组件控制这数据）。只存在内部state的数据被称作**非受控的**（因为父组件不能直接改变它）。

最常见的错误是将两者搞混了。当一个派生状态同时被`setState`更新的时候，数据就失去了单一的事实来源。上面提到的[加载数据](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html#fetching-external-data-when-props-change)的例子看上去是类似的，但在一些关键的地方是有区别的。在例子中，每当source属性变化，loading状态**必定**会被覆盖。反过来，状态要么在props变化的时候被覆盖，要么由组件自己管理。（译注：可理解为同时只有单一的真实来源）

当任何一个限制被改变的时候就会发生问题，下面举了两个典型的例子。

### 反例：无条件地复制props到state

一个常见的误解是`getDerivedStateFromProps`和`componentWillReceiveProps`只会在props“改变”的时候调用。这些生命周期会在任何父组件发生render的时候调用，不管props是否真的改变。因此，使用这些周期去**无条件**地覆盖state是不安全的。**这样做会使得state丢失更新**。

我们来演示一下这个问题。这是一个`邮件输入`组件，它“映射”了email属性到state：

```js
class EmailInput extends Component {
  state = { email: this.props.email };

  render() {
    return <input onChange={this.handleChange} value={this.state.email} />;
  }

  handleChange = event => {
    this.setState({ email: event.target.value });
  };

  componentWillReceiveProps(nextProps) {
    // 这样会抹去所有的内部状态更新!
    // 不要这样做.
    this.setState({ email: nextProps.email });
  }
}
```

看上去这个组件好像没问题。state被props中的值初始化，然后随着`<input>`的输入而更新。但是如果父组件发生render，我们在`<input>`中输入的东西都会消失！（[参见这里的demo](https://codesandbox.io/s/m3w9zn1z8x)）即使我们在重置前比较`nextProps.email !== this.state.email`也会如此。

在这个简单的例子中，添加`shouldComponentUpdate`去限制只在props中的email发生改变时才去重新render能解决这个问题。但在实际中，组件通常会接受很多的props。你无法避免其他的属性发生改变。函数和对象属性通常是内联创建的，这让我们很难去实现判断是否发生了实质性的变化。[这里有一个demo来说明](https://codesandbox.io/s/jl0w6r9w59)。因此，`shouldComponentUpdate`最好只是用来优化性能，而不是去确保派生状态的正确性。

希望现在我们能弄清楚为什么**无条件地复制props到state是坏主意**。在查看可能的解决方案前，我们先来看一个相关的例子：如果我们只在email属性发生改变的时候更新state呢？

### 反例：props改变时覆盖state

继续上面的例子，我们可以通过只在`props.email`改变时更新state来避免意外地覆盖已有的state。

```js
class EmailInput extends Component {
  state = {
    email: this.props.email
  };

  componentWillReceiveProps(nextProps) {
    // 只要props.email改变, 更新state.
    if (nextProps.email !== this.props.email) {
      this.setState({
        email: nextProps.email
      });
    }
  }
  
  // ...
}
```

> 尽管上面的例子中使用的是`componentWillReceiveProps`，但这个反例同样的也对`getDerivedStateFromProps`适用

我们刚迈进了一大步。现在我们的组件只会在props真正改变的时候覆盖掉我们输入的东西了。

当然这里还是有一个微妙的问题。想象一个密码管理app使用了如上的输入组件。当切换两个不同的账号的时候，如果这两个账号的邮箱相同，那么我们的重置就会失效。因为对于这两个账户传入的email属性是一样的。（译注：好比你切换了一份数据源，但是这两份数据中的email是相等的，于是预期应该被重置的输入框没有被重置）[查看demo](https://codesandbox.io/s/mz2lnkjkrx)

这个设计从根本上是有缺陷的，但却是很容易犯的错误。（[我自己也曾犯过](https://twitter.com/brian_d_vaughn/status/959600888242307072)）幸运的是有两个更好的可选方案。关键在于**对于任何数据，你需要选择一个作为其真实来源的组件，并且避免在其他组件中复制它**。我们来看看下面的解决方案。

## 更好的方案

### 推荐：完全受控组件

一个解决上述问题的方案是完全移除我们组件中的state。如果email仅作为属性存在，我们就不需要担心和state的冲突。我们甚至可以把`EmailInput`作为一个更轻量级的函数组件：

```js
function EmailInput(props) {
  return <input onChange={props.onChange} value={props.email} />;
}
```

此举简化了我们组件的实现，但是如果我们仍然想保存一份输入的草稿值呢，这时候需要父组件手动来实现了，[请看demo](https://codesandbox.io/s/7154w1l551)

### 推荐：用key标识的完全不受控组件

另一个可选方案是我们的组件完全控制eamil的“草稿”状态。在这里，我们的组件仍然接受props中的属性用来初始值，但是它会忽略后续属性的变化。

```js
class EmailInput extends Component {
  state = { email: this.props.defaultEmail };

  handleChange = event => {
    this.setState({ email: event.target.value });
  };

  render() {
    return <input onChange={this.handleChange} value={this.state.email} />;
  }
}
```

为了在切换不同内容时重置输入值（像上面提到的密码管理器），我们可以使用React的`key`属性。当`key`改变，React会[重新创建一个新的组件而不是更新它](https://reactjs.org/docs/reconciliation.html#keys)。Key一般用在动态列表，但是在这也是很有用的。这里当选择一个新用户的时候，我们用用户ID去重新创建这个email输入组件：

```js
<EmailInput
  defaultEmail={this.props.user.email}
  key={this.props.user.id}
/>
```

每当ID改变，`EmailInput`会被重新创建然后state会被重置为最新的`defaultEmail`值。（[点这查看demo](https://codesandbox.io/s/6v1znlxyxn)）其实你无需为每一个输入框添加一个key。对整个表单添加一个`key`会显得更有用。每当key改变，表单的所有组件都会被重建且被赋上干净的初始值。

大多数情况，这是重置state最好的办法。

> 重新创建组件听上去会很慢，但其实对性能的影响微乎其微。如果组件具有很多更新上的逻辑，则使用key甚至可以更快，因为该子树的差异得以被绕过。

#### 替代方案1：通过ID属性重置非受控组件

如果因为某些原因无法使用`key`（比如组件初始化的代价很高），一个可行但笨重的办法是在`getDerivedStateFromProps`监听“userID”的改变。

```js
class EmailInput extends Component {
  state = {
    email: this.props.defaultEmail,
    prevPropsUserID: this.props.userID
  };

  static getDerivedStateFromProps(props, state) {
    // Any time the current user changes,
    // Reset any parts of state that are tied to that user.
    // In this simple example, that's just the email.
    if (props.userID !== state.prevPropsUserID) {
      return {
        prevPropsUserID: props.userID,
        email: props.defaultEmail
      };
    }
    return null;
  }

  // ...
}
```

如果我们这样选择，这也提供了仅重置部件的内部状态的灵活性([这里查看demo](https://codesandbox.io/s/rjyvp7l3rq))

> 上面的例子对于componentWillReceiveProps也是一样的

#### 替代方案2：使用实例方法重置非受控组件

在极少情况，你可能需要在没有合适的ID作为key的情况下重置state。一个办法是把key设为随机值或者递增的值，在你想要重置的时候改变它。另一个可行方案是暴露一个实例方法命令式地去改变内部的state：

```js
class EmailInput extends Component {
  state = {
    email: this.props.defaultEmail
  };

  resetEmailForNewUser(newEmail) {
    this.setState({ email: newEmail });
  }

  // ...
}
```

父表单组件就可以[通过ref去调用这个方法](https://reactjs.org/docs/glossary.html#refs)，（[点击这里查看demo](https://codesandbox.io/s/l70krvpykl)）

Ref这这类场景中挺有用，但我们推荐你尽量谨慎地去使用。即使在这个例子中这种方法也是不理想的，因为会造触发两次render。

## 关于缓存记忆（memoization）

我们也见到过用派生状态来确保`render`中计算量较大的值仅在输入改变的时候重新计算。这个技术叫做[memoization](https://en.wikipedia.org/wiki/Memoization)。

使用它作缓存记忆不一定是不好的，但通常不是最佳解决方案。管理派生状态有一定的复杂度，这个复杂度还会随着额外的属性而增加。比如，如果我们给组件添加了第二个派生字段，那么我们需要分别跟踪这两个字段的变化。

我们来看一个例子，我们把一个列表传入这个组件，然后它需要按用户的输入筛选显示出匹配的项。我们可以使用派生状态去保存筛选后的列表。

```js
class Example extends Component {
  state = {
    filterText: "",
  };

  // *******************************************************
  // NOTE: 这个例子不是我们推荐的做法
  // 推荐的方法参见下面的例子.
  // *******************************************************

  static getDerivedStateFromProps(props, state) {
    // 每当列表数组或关键字变化时筛选列表.
    // 注意到我们需要储存prevPropsList和prevFilterText来监听变化.
    if (
      props.list !== state.prevPropsList ||
      state.prevFilterText !== state.filterText
    ) {
      return {
        prevPropsList: props.list,
        prevFilterText: state.filterText,
        filteredList: props.list.filter(item => item.text.includes(state.filterText))
      };
    }
    return null;
  }

  handleChange = event => {
    this.setState({ filterText: event.target.value });
  };

  render() {
    return (
      <Fragment>
        <input onChange={this.handleChange} value={this.state.filterText} />
        <ul>{this.state.filteredList.map(item => <li key={item.id}>{item.text}</li>)}</ul>
      </Fragment>
    );
  }
}
```

这里的实现避免了`filteredList`不必要的重新计算。但其实过于复杂了，因为它需要分别跟踪和监听props和state的变化来正确地更新筛选列表。这个例子中，我们可以使用`PureComponent`来简化操作然后把筛选操作放到render方法中去：

```js
// PureComponents只会在至少state和props中有一个属性发生变化时渲染.
// 变化是通过引用比较来判断的.
class Example extends PureComponent {
  // State only needs to hold the current filter text value:
  state = {
    filterText: ""
  };

  handleChange = event => {
    this.setState({ filterText: event.target.value });
  };

  render() {
    // 只有props.list 或 state.filterText 改变时才会调用.
    const filteredList = this.props.list.filter(
      item => item.text.includes(this.state.filterText)
    )

    return (
      <Fragment>
        <input onChange={this.handleChange} value={this.state.filterText} />
        <ul>{filteredList.map(item => <li key={item.id}>{item.text}</li>)}</ul>
      </Fragment>
    );
  }
}
```

上面的方法比派生状态的版本更简单干净。然而这也不是个好方法，因为对于长列表来说可能比较慢，而且`PureComponent`无法阻止其他属性改变造成的render。为了应对这种情况我们引入了一个memoization辅助器来避免多余的筛选。

```js
import memoize from "memoize-one";

class Example extends Component {
  // State只需要去维护目前的筛选关键字:
  state = { filterText: "" };

  // 当列表数组或关键字变化时重新筛选
  filter = memoize(
    (list, filterText) => list.filter(item => item.text.includes(filterText))
  );

  handleChange = event => {
    this.setState({ filterText: event.target.value });
  };

  render() {
    // 计算渲染列表时，如果参数同上次计算没有改变，`memoize-one`会复用上次返回的结果
    const filteredList = this.filter(this.props.list, this.state.filterText);

    return (
      <Fragment>
        <input onChange={this.handleChange} value={this.state.filterText} />
        <ul>{filteredList.map(item => <li key={item.id}>{item.text}</li>)}</ul>
      </Fragment>
    );
  }
}
```

这样更简单地实现了和派生状态一样的功能！

## 总结

在实际应用中，组件通常既包含受控与非受控的元素。这没问题，如果每个数据都有清晰的真实来源，你就可以避开上面提到的反例。

`getDerivedStateFromProps`的用法值得被重新思考，因为他是一个拥有一定复杂度的高级特性，我们应该谨慎地使用。

> 原文链接：[You Probably Don't Need Derived State
](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)

