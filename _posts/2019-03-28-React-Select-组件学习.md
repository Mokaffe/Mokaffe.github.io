---
layout:     post
title:      React-Select 组件学习
subtitle:   组件React-Select的使用记录
date:       2019-03-28
author:     Mokaffe
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - react-select
---

> 学习资料
- [react-select github](https://github.com/jedwatson/react-select)
- [v2 - display icon in selected option](https://github.com/JedWatson/react-select/issues/2553)
- Github React-Select Demo： [Mokaffe](https://github.com/Mokaffe)/**[react-tutorial](https://github.com/Mokaffe/react-tutorial)**


### React-Select 介绍
[React-Select](https://github.com/JedWatson/react-select) 是 Select Component for React.js.
可在 [react-select.com](https://www.react-select.com/) 网站看在线demo和使用文档。
> It represents a whole new approach to developing powerful React.js components that just work out of the box, while being extremely customizable.

现在是React-Select Version 2 版本，它代表了一种开发功能强大的React.js组件的全新方法，这些组件只需开箱即用，同时具有极高的可定制性。
> See our [upgrade guide](https://react-select.com/upgrade-guide) for a breakdown on how to upgrade from v1 to v2.
The old docs and examples will continue to be available at [v1.react-select.com](https://v1.react-select.com/).

如果之前是使用react-select v1 版本，可以查看upgrade guide去进行升级v2操作，v1版本的文档和例子在 [v1.react-select.com](https://v1.react-select.com/).

React-Select 的一些特色：
> Improvements include:
> *   Flexible approach to data, with customisable functions
> *   Extensible styling API with [emotion](https://emotion.sh/)
> *   Component Injection API for complete control over the UI behaviour
> *   Controllable state props and modular architecture
> *   Long-requested features like option groups, portal support, animation, and more

### React-Select 使用

#### 一些常用的props
- `autoFocus` - focus the control when it mounts（这个props是boolean 值，当react-select组件mount后，自动选中到select 输入框中）
- `className` - apply a className to the control
- `classNamePrefix` - apply classNames to inner elements with the given prefix( This is useful when styling via CSS classes instead of the Styles API approach.)
- `isDisabled` - disable the control
- `isMulti` - allow the user to select multiple values
- `isSearchable`- allow the user to search for matching options
- `name` - generate an HTML input with this name, containing the current value （会在select组件下面生成一个hide的input，input里面的value是当前select选中的值）
- `onChange` - subscribe to change events
- `options` - specify the options the user can select from
- `placeholder` - change the text displayed when no option is selected
- `value` - control the current value

参考docs [props documentation](https://www.react-select.com/props) 获取更多的信息。

**上诉常见的props的效果图** 

![`autoFocus ` 效果图.png](https://upload-images.jianshu.io/upload_images/5013098-2ea346b1833268a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![className.png](https://upload-images.jianshu.io/upload_images/5013098-03b2420dfdfc9032.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![classNamePrefix.png](https://upload-images.jianshu.io/upload_images/5013098-0cb0d508f343c939.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![name.png](https://upload-images.jianshu.io/upload_images/5013098-92e441495c09b354.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![onChange.png](https://upload-images.jianshu.io/upload_images/5013098-611131ffbe99674d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Controllable Props（可控制的props)
>You can control the following props by providing values for them. If you don't, react-select will manage them for you.（你可以通过为它们提供值去控制下面这些props，如果不这样做，react-select 将帮你管理它们。）
> - `value` / `onChange` - specify the current value of the control
> - `menuIsOpen` / `onMenuOpen` / `onMenuClose` - control whether the menu is open
> - `inputValue` / `onInputChange` - control the value of the search input (changing this will update the available options)

Question:
- Control 是react-select 中的哪个组件？
- menu 是react-select 中的哪个组件？
- search input 是react-select 中的哪个组件？
这些问题可以在参考文档[replaceable component architecture](https://react-select.com/components)部分找到答案，但是你想特别明确的知道这些是哪个部分，需要自己一一验证。或者可以使用`classNamePrefix`这个props，可以去审查元素，能够知道react-select中组件的分层。

> If you don't provide these props, you can set the initial value of the state they control:(如果你没有提供这些props，你可以设置这些props掌控的状态的初始值)
> - `defaultValue` - set the initial value of the control
> - `defaultMenuIsOpen` - set the initial open value of the menu
> - `defaultInputValue` - set the initial value of the search input

#### 高可定制
查看下面文档去使用一些定制功能:
*   [Customising the styles](https://www.react-select.com/styles)
*   [Using custom components](https://www.react-select.com/components)
*   [Using the built-in animated components](https://www.react-select.com/home#animated-components)
*   [Creating an async select](https://www.react-select.com/async)
*   [Allowing users to create new options](https://www.react-select.com/creatable)
*   [Advanced use-cases](https://www.react-select.com/advanced)


### 使用react-select时遇到需要修改默认样式

#### 单选后鼠标光标在输入框的最前面
单选的时候，如果想要在输入框清除选中的数据，光标在输入框的最前面，而不是在选中数据的后面。UX期希望将光标移到选中值的后面。

![image.png](https://upload-images.jianshu.io/upload_images/5013098-f00b20b9acc80c85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5013098-b50c503458ef1765.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/5013098-124791ec00152798.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

参考文档[Styles](https://react-select.com/styles)部分.
- clearIndicator
- container
- control
- dropdownIndicator
- group
- groupHeading
- indicatorsContainer
- indicatorSeparator
- input
- loadingIndicator
- loadingMessage
- menu
- menuList
- menuPortal
- multiValue
- multiValueLabel
- multiValueRemove
- noOptionsMessage
- option (这个是控制下拉列表中单个选项的样式)
- placeholder
- singleValue
- valueContainer

#### 定制下拉列表的样式
react-select 默认的下拉列表的选项都是字符串，设计稿中的选项是由用户名和email组合成上下层级的选项，所以需要定制Option这个组件。
![image.png](https://upload-images.jianshu.io/upload_images/5013098-f0a3094775c62d18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/5013098-3e4105368063ce34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/5013098-1a0779cd5c4a72dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


参考解决方法[v2 - display icon in selected option](https://github.com/JedWatson/react-select/issues/2553)
参考文档[replaceable component architecture](https://react-select.com/components)部分
React-select 允许你用自己的组件替换内部的组件，可替换的组件有：
- ClearIndicator
- Control
- DropdownIndicator
- Group
- GroupHeading
- IndicatorsContainer
- IndicatorSeparator
- Input
- LoadingIndicator
- Menu
- MenuList
- LoadingMessage
- NoOptionsMessage
- MultiValue
- MultiValueContainer
- MultiValueLable
- MultiValueRemove
- Option
- Placeholder
- SelectContainer
- SingleValue
- ValueContainer

