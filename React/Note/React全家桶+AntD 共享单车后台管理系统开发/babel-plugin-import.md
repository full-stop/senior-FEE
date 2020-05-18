# babel-plugin-import

## introduction

babel-plugin-import 是阿里 `Ant` 团队开发的一个 `babel` 插件，它可以实现代码的按需打包。

例如：

``` jsx
import {Button} from 'antd';
 ↓ ↓ ↓ ↓ ↓ ↓
var _button = require('antd').button;
```

如果上面的代码没有进行任何处理，那么对于 `webpack` 来说它需要将整个 `antd` 库都打入到编译文件中，这样很多没有使用到的组件都会被引入，从而增加代码文件的体积。

使用了 `babel-plugin-import` 库，那么对于上面的代码它会进行如下的处理：
**{ "libraryName": "antd"}**

``` jsx
import { Button } from 'antd';
ReactDOM.render(<Button>xxxx</Button>);
 
      ↓ ↓ ↓ ↓ ↓ ↓
 
var _button = require('antd/lib/button');
ReactDOM.render(<_button>xxxx</_button>);
``` 
而如果指定了 `style` 属性的配置，则会只打包使用到的组件样式文件。
**{ "libraryName": "antd", style: "css" }**

```jsx
import { Button } from 'antd';
ReactDOM.render(<Button>xxxx</Button>);
 
      ↓ ↓ ↓ ↓ ↓ ↓
 
var _button = require('antd/lib/button');
require('antd/lib/button/style/css');
ReactDOM.render(<_button>xxxx</_button>);
```

`style` 属性为 `css` 则只会导入对应组件中的 css 文件，这样的文件都是已经编译好的样式文件，如果为了尽可能的减少样式文件的打包体积，可以设置 `style:true` ，从而导入对应组件的CSS源文件，例如可能是 less文件或scss文件。

**{ "libraryName": "antd", style: true }**

``` jsx
import { Button } from 'antd';
ReactDOM.render(<Button>xxxx</Button>);
 
      ↓ ↓ ↓ ↓ ↓ ↓
 
var _button = require('antd/lib/button');
require('antd/lib/button/style');
ReactDOM.render(<_button>xxxx</_button>);
```

`babel-plugin-import` 插件不仅可以用于阿里的 antd 库，它也可以支持自定义的库，下面的一些属性与方法对于自定义库的按需导入非常有用：

* **camel2DashComponentName：** `Boolean` 是否开启对组件名的驼峰解析。
* **libraryDirectory：** `String` 自定义组件的目录路径，默认 `lib` . 
* **styleLibraryDirectory：** `String` 自定义组件样式的目录路径。

最后如果想自定义组件的引入路径或样式文件的路径以及过滤某些没有样式的组件，则可以使用以下属性，它们支持传入一个函数方法作为值。

* **customStyleName**
* **customName**
* **style**

## Usage

**install**

``` text
npm install babel-plugin-import --save-dev
```

**configure**
在 `.babelrc` 或者 `babel-loader` 

``` text
{
  "plugins": [["import", options]]
}
```

其中 `options` 默认就是 `{ "libraryName": "antd"}` 
