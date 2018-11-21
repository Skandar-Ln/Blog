---
title: rc-form之动态表单
date: 2018-02-05 16:20:28
tags: react antd
---
### 背景
在项目中用到了ant design组件库（antd），由于antd的form组件依赖了[rc-form][1]这个高阶react的表单组件，于是我们是隔着antd这一层调用了rc-form的接口，但是对于rc-form的细节，或者说对于它的接口并不是完全了解。加上平时会出现对动态form之类使用不正常导致bug或者有更方便的接口没有使用上等情况出现。所以这里对rc-form这一组件做出了一些调研。毕竟，对于web应用来说总是绕不过表单这个复杂的东西。

### 简述
在getFieldDecorator的时候都发生了什么？
``` js
<FormItem label="account">
    {getFieldDecorator('account')(
        <Input name="account" />
    )}
</FormItem>
```
对于一个form的表单项来说，它在render的时候到底发生了什么。
如果对一个固定位置的表单项来说，它会先从
``` js
getFieldDecorator(name, fieldOption) {
    const props = this.getFieldProps(name, fieldOption);
    return (fieldElem) => {
        const fieldMeta = this.fieldsStore.getFieldMeta(name);
        const originalProps = fieldElem.props;
        // ...
        fieldMeta.originalProps = originalProps;
        fieldMeta.ref = fieldElem.ref;
        return React.cloneElement(fieldElem, {
            ...props,
            ...this.fieldsStore.getFieldValuePropValue(fieldMeta),
        });
    };
}
```

[1]: https://github.com/react-component/form