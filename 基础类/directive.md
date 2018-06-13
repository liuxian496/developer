# $$directive 用来封装控件的Angular指令
$$directive是以jQuery为基础的构建的一个工厂方法，使用它为控件创建指令。$$directive提供指令module管理、依赖注入、模板解析，调用控件公共方法等功能。用来优化，统一AngularJS指令的编写。

## 指令实例
使用$$directive创建指令实例

Parameters:
- name:指令名称。类型：string。
- prototype:指令原型。类型：object。

## 指令名称
指令名称是一个字符串，由三部分组成，分别是module name 、widget name 、directive后缀。各部分用.号连接。
- module name是一个小写的字符串，根据这个名字创建一个AngularJS module，module name相同的指令，会自动配置到该module中。相当于指令的namespace。
- widget name使用驼峰命名法，根据这个名字创建一个访问控件公共方法的访问器；创建一个用来注册指令的方法；创建自定义标签、属性。

示例：

```
ui.combobox.directive
ss.matirxType.directive
```
## 指令标签
根据指令名称创建一个自定义html标签。 创建过程，符合以下规则：
1. 去掉directive后缀
2. 将.号转换成-
3. 将驼峰命名法转换成-链接

示例：

```
<ui-combobox></ui-combobox>
<ss-matirx-type></ss-matirx-type>

```
## 指令参数
根据widget name 创建一个html属性（attribute），这个属性用来设置控件参数，称为指令参数。值是一个字符串。会在全局作用域（window）下解析这个字符串。符合以下规则

1. 将驼峰命名法转换成-链接
2. 使用时请加上data-前缀

示例:

```
<ui-combobox data-combobox="{as:'vm', disabled:'disabled', dataTextField:'color', dataValueField:'color', items:'colors', selectedItem:'selectedColorText'}"></ui-combobox>

<ss-matirx-type data-matirx-type="{as:'vm',rows:'rows',columns:'columns',labelComment:'labelComment',comment:'comment'}"></ss-matirx-type>

```
# module管理

## aui指令集
aui控件库的指令，会注册到名称为aui的module上，这个module称为aui指令集

```
angular.module('aui');

```

## 项目指令集
项目的指令，会注册到名称为[module name].widgets的module上，这个module称为[module name]指令集

```
//ss.matirx.type.directive.js
$$directive("survey.matirxType.directive",{});

//ss
angular.module('ss.directive');
```

## 配置指令集

app.module.js

```
//启动module 使用ng-app挂载到根元素
angular.module('app.angular', []);
angular.module('app', ['aui', 'app.angular']);
```

common.services.js


```
(function(){
    angular.module('common.services', []).factory('logger', logger);
    logger.$inject = ['$log'];
    function logger($log) {
        var service = {
            log: $log.log
        }

        return service;
    }
})();
```

app.angular.module.js

```
(function(){
    //用来配置服务，指令，常量，filter的module
    angular.module('combobox.services', []);
    angular.module('app.angular', ['combobox.services']);
})();
```

combobox.services.js

```
angular.module('combobox.services', ['common.services']).factory('comboboxDataService', comboboxDataService);

function comboboxDataService() {
    var service = {
        x: 100
    }

    return service;
}

```

combobox.controller.js

```
(function(){
    $$page("demo.combobox.controller", {
        //controller logic
    });
    angular.module('app.angular').controller('ComboboxController', $$page.demo_combobox_controller);
    //注入服务
    $$page.demo_combobox_controller.$inject = ['logger', 'comboboxDataService'];
})();
```

# 初始化参数

## _inject
设置一个数组，数组的每一项是一个字符串。表示为指令注入的服务，类型：array
$parse和$scope服务会默认注入，请不要注入这两个服务。

默认值：null

## _template

设置一个符合html语法的字符串，该字符串作为指令的模板。类型：string

默认值：null

## _templateUrl

设置url，表示指令使用的html模板。类型：url

默认值：null

# 调用控件公共方法
name属性在指令参数中设置,值是一个字符串，根据这个字符串在数据上下文中创建一个属性。属性的值是一个function，使用这个函数调用控件的API方法

view:
```
<!-- 将name设置成productGrid -->
<ui-datagrid data-datagrid="{as:'vm', name:'productGrid', width: 440, height:200, items:'records', itemsSourceChanged:'onItemsSourceChanged', viewModel:{isSelectAllChecked:'isSelectAllChecked', onSelectAllClick:'onSelectAllClick', onCheckboxClick:'onCheckboxClick'}}"></ui-datagrid>
```
controller:

```
//获取所有isChecked属性的指是true的行
_onGetSelectedItems: function () {
        var
            i = 0,
            items = this._vm.productGrid('getRowsByAttribute', 'isChecked', true),
            c = items.length,
            names = '';

        for (; i < c; i++) {
            names += ' "' + items[i].productName + '"';
            if (c > 1 && i < c - 1) {
                names += ' and ';
            }
        }

        alert('Selected products name is' + names + ' .');
    }
```

# 添加自定义事件

1.获取在数据上下文中回调函数的名称

```
_initMembers: function () {
    var args = this._args;
    
    //itemsSourceChange事件在数据上下文中的名称
    this._itemsSourceChangedName = args.itemsSourceChanged;
}
```

2. 在_initWidget中初始化控件实例时，注册selectionChanged事件。

```
_initWidget: function (base) {
    //base:指令实例
    var data = { self: this, base: base };

    this._ele
        .on("selectionChanged", data, base._onSelectionChanged)
        .on("shown", data, base._onShown)
        .combobox(this._args);
}
```

3. 在事件回调中，使用_trigger方法调用上下文中的关联事件

```
_onItemsSourceChanged: function (e, args) {
    var self = e.data.self;
    e.data.base._trigger(e, args, self._itemsSourceChangedName);
}
```

## _trigger

- e:jQuery事件参数。类型：string。
- args:控件自定义事件参数。类型：object。
- name:回调函数在数据上下文中的名称。类型：function。

# Interface

创建指令时，按顺序实现以下接口。

## 1. _initMembers
在该接口中，需要为实例创建属性，记录绑定，监听属性的名称。之后会使用这些属性创建监听和获取数据。

### 默认成员

成员变量以下划线开头（private）。以下5个属性由基类创建，可以直接使用

- _args：控件初始化参数。类型object
- _ele：控件宿主对应的jQuery对象。类型：jQuery
- _vm：控件绑定的数据上下文。类型：object
- _scope：$scope服务
- _isAs：表示是否使用了Controller As语法。类型：boolean。true表示使用了Controller As语法。不要对该属性赋值。
- 每一个注入的服务，会生成一个以下划线开头的变量


```
_initMembers: function () {
    var args = this._args;

    if (args.selectedItem == undefined) {
        $.error("Mapping 'selectedItem' is not available！");
    }
    //combobox selecteItem的监听名称
    this._watchSelectedItem = this._selectedItemName = args.selectedItem;

    this._watchItems = this._itemsName = args.items;

    this._watchDisabled = this._disabledName = args.disabled;

    //表示是否正在监听
    this._isSelectedWatched = false;

    //hidden事件在数据上下文中的名称
    this._hiddenName = args.hidden;
    //itemsSourceChange事件在数据上下文中的名称
    this._itemsSourceChangedName = args.itemsSourceChanged;
    //selectedChanged事件在数据上下文中的名称
    this._selectedChangedName = args.selectionChanged;
    //shown事件在数据上下文中的名称
    this._shownName = args.shown;

    this._isSetDisabled = typeof args.disabled == "string" && !!args.disabled;
}
```


## 2. _convertAs

在该接口中完成两件事情：

- 为需要监听（$watch）的属性实现Controller As语法
- 从AngularJS的数据上下文中获取控件参数


```
_convertAs: function (transform) {
    var args = this._args;
    if (typeof args.as == "string" && !!args.as) {
        this._watchSelectedItem = transform(args.as, this._selectedItemName);
        this._watchItems = transform(args.as, this._itemsName);

        if (this._isSetDisabled) {
            this._watchDisabled = transform(args.as, this._disabledName);
        }
    }
    
    args.selectedItem = this._vm[this._selectedItemName];
}
```

## 3. _getTemplate

在该接口中，获取控件子模板。

Parameters:

- base:指令实例。类型：object。
- trim:用来去除多余的空格。类型：function。

```
_getTemplate: function (base, trim) {
    var
        ele = this._ele,
        args = this._args;

    //获取可选项模板
    args.itemTemplate = trim(ele.children('p').children('item-template').html());
    //获取选中项模板
    args.selectedTemplate = trim(ele.children('p').children('selected-template').html());
    //create new模板
    args.createTemplate = trim(ele.children('p').children("create-template").html());
}
```

## 4. _initWidget

在该接口中，初始化控件实例。清除指令标签和指令参数

Parameters:

- base:指令实例。类型：object。


```
_initWidget: function (base) {
    var data = { self: this, base: base };

    this._ele
        .on("destroy", data, base._onDestroy)
        .on("hidden", data, base._onHidden)
        .on("itemsSourceChanged", data, base._onItemsSourceChanged)
        .on("selectionChanged", data, base._onSelectionChanged)
        .on("shown", data, base._onShown)
        //清楚指令标签
        .removeAttr("data-aui-combobox")
        //清楚指令参数
        .removeAttr("data-combobox")
        .combobox(this._args);
}
```

## 5. _addWatcher
在该接口中，注册AngularJS监听。

Parameters:

- base:指令实例。类型：object。


```
_addWatcher: function (base) {
    //itemsSource需要先与selectedItem监听
    base._addItemsWatcher.call(this);
    base._addSelectedItemWatcher.call(this);
    if (this._isSetDisabled) {
        base._addDisabledWatcher.call(this);
    }
}
```

# code

```
(function ($) {
    "use strict";
    var uuid = -1, body = $('html'), _statics, _directive, _interface, auiEventDirectives = {};

    if (window.$$directive) {
        $$.error('$$directive is used by other code.Please resolve conflict.');
    };

    if (window.angular) {
        angular.module('aui', []);
    } else {
        $$.error('angular is undefined.');
    }

    /**
     * @param {string} name 命名空间+实例名称
     * #param {object} prototype 原型
     */
    window.$$directive = function (name, prototype) {
        init(name, prototype);
    }

    _directive = ["_trigger", "_apply", "_setValue", "_geValue", "_destroy", '_watchers', '_watcherRegister', '_isDestroyed'];
    _interface = ['_initMembers', '_convertAs', '_getTemplate', '_initWidget', '_addWatcher', '_widgetName', '_namespace'];

    angular.forEach(
      'click dblclick mousedown mouseup mouseover mouseout mousemove mouseenter mouseleave keydown keyup keypress submit focus blur copy cut paste'.split(' '),
      function (eventName) {
          var directiveName = 'aui' + firstUpperCase(eventName);
          auiEventDirectives[directiveName] = ['$parse', '$rootScope', function ($parse, $rootScope) {
              return {
                  restrict: 'A',
                  compile: function ($element, attr) {
                      var fn = $parse(attr[directiveName], /* interceptorFn */ null, /* expensiveChecks */ true);
                      return function ngEventHandler(scope, element) {
                          element.on(eventName, function (event) {
                              var callback = function () {
                                  fn(scope, { $event: event });
                              };
                              callback();
                          });
                      };
                  }
              };
          }];
      }
    );
    angular.module('aui').directive(auiEventDirectives);

    function getArgs(name, scope, args) {
        //求正则
        var
            i = 0,
            args = [],
            list = name.replace(/\(|\)/g, " ").split(" ")[1],
            c;

        if (typeof list == "string") {
            list = list.split(',');
        } else {
            list = [];
        }

        c = list.length;

        for (; i < c; i++) {
            args[i] = scope[list[i]];
        }

        return args;
    }

    function getParent(linker, name) {
        var
            names,
            i = 0,
            c,
            parent = linker._vm;
        if (name) {
            names = name.split('(')[0].split('.');
            c = names.length - 1;
            if (parent === null) {
                $$.error('Can not find property "' + names[0] + '" .Please check your binding syntax.');
            }
            for (; i < c; i++) {
                parent = parent[names[i]];
            }
        }
        if (parent === undefined) {
            return getParent({
                _vm: linker._scope.$parent,
                _scope: linker._scope.$parent
            }, name);
        } else {
            return {
                parent: parent,
                name: names && names[c]
            }
        }
    }

    page.prototype = {
        //调用数据上下文中绑定到控件事件的回调。
        _trigger: function (e, args, name, isPropagationStopped) {
            var
                self = e.data,
                temp,
                callback;
            if (isPropagationStopped && args.type != self._widgetName) {

            } else if (typeof name == 'string' && !!name) {
                temp = getParent(self, name);
                callback = temp.parent[temp.name];
                if ($.isFunction(callback)) {
                    e.data = undefined;
                    args && (args.element = undefined);

                    return callback.apply(self._vm, [e, args].concat(getArgs(name, self._scope)));
                } else {
                    $$.error('"' + name + '" is not a function.Please check your binding syntax.');
                }
            }
        },
        _destroy: function (name) {
            var
                temp,
                c = this._watchers.length;
            this._isDestroyed = true;
            if (typeof name == "string" && !!name) {
                temp = getParent(this, name);
                delete temp.parent[temp.name];
            }
            //destroy angular watcher
            for (var i = 0; i < c; i++) {
                if ($.isFunction(this._watchers[i])) {
                    try {
                        this._watchers[i]();
                    } catch (e) {
                        $$.log("Deregister watch failed.");
                    }
                }
            }
        },
        _setValue: function (name, value) {
            var temp;

            if (name) {
                temp = getParent(this, name);
                temp.parent[temp.name] = value;
            }
        },
        _geValue: function (name) {
            var temp;

            if (name) {
                temp = getParent(this, name);
                return temp.parent[temp.name];
            }
        },
        _watcherRegister: function (watcher) {
            if ($.isArray(watcher)) {
                var c = watcher.length;
                for (var i = 0; i < c; i++) {
                    this._watchers.push(watcher[i]);
                }
            } else {
                this._watchers.push(watcher);
            }
        },
        _watchers: [],
        //为指令添加公共属性
        _initMembers: $.noop,
        _getTemplate: $.noop,
        //支持 controller as语法
        _convertAs: $.noop,
        _initWidget: $.noop,
        _addWatcher: $.noop
    }

    //page实例的构造函数
    function page() {
        if (!(this instanceof page)) {
            return new page();
        }
    }
    //link实例的构造函数
    function linker() {
        if (!(this instanceof linker)) {
            return new linker();
        }
    }

    //创建一个方法，用来访问控件实例的公共API
    function vistor(widget) {
        var
            self = this,
            context = "scope",
            temp,
            args = this._args;
        temp = getParent(this, args.name);
        if (typeof args.name == "string" && !!args.name) {
            if (temp.parent.hasOwnProperty(temp.name)) {
                $$.error(args.name + ' is already exist.');
            }

            temp.parent[temp.name] = function () {
                if (arguments[0] == "option") {
                    $$.error('call method "option" is not allowed.')
                }
                if (typeof self._widgetName == 'string' && !!self._widgetName) {
                    widget = self._widgetName;
                }
                return self._ele[widget].apply(self._ele, arguments);
            };
        }
    }

    function transform(as, name) {
        if (name.indexOf(".") == -1 && name != undefined) {
            name = as + '.' + name;
        }

        return name;
    }

    /**
     * 获取模板
     * @param {jQuery} ele 承载模板的jQuery对象
     */
    function trimTemplate(temp) {
        return temp && temp.trim();
    }

    function filter(keys, obj) {
        var i = 0, c = keys.length;

        for (; i < c; i++) {
            delete obj[keys[i]];
        }
    }

    function firstUpperCase(str) {
        return str.replace(/(\w)/, function (v) {
            return v.toUpperCase()
        });
    }

    function setLinkArgs(directive, link, args) {
        var
            i = 0,
            c = args.length - 1,
            inject = directive[link].$inject,
            value = {};

        for (; i < c; i++) {
            directive['_' + inject[i].replace('$', '')] = args[i];
        }
        value.base = directive;

        return value;
    }

    function addLinker(directive, widget, link) {
        if (!$.isFunction(directive[link])) {
            directive[link] = function () {
                var args = setLinkArgs(directive, link, arguments);
                return {
                    restrict: 'EA',
                    template: directive._template,
                    templateUrl: directive._templateUrl,
                    replace: true,
                    transclude: true,
                    link: function (scope, elem, attrs) {
                        args._vm = scope;
                        args._ele = $(elem);
                        args._scope = scope;
                        args._attrs = attrs;
                        bridge(args.base, widget, link);
                        initLinker.call(linker(), args);
                    }
                }

            };
        }
    }

    function bridge(base, widget, link) {
        var
            i = 0,
            c = _interface.length;
        linker.prototype = {};
        $.extend(true, linker.prototype, base);
        for (; i < c; i++) {
            delete linker.prototype[_interface[i]];
        }
        delete linker.prototype[widget];
        delete linker.prototype[link];
    }

    function addInjecter(directive, widget, link, name) {
        //需要注入的服务
        var
            inject = directive._inject,
            dName = directive._namespace + firstUpperCase(widget);

        if (!$.isArray(inject)) {
            inject = [];
        }
        if (!$.isFunction(directive[widget])) {
            directive[widget] = function (app) {
                var self = this;
                self[link].$inject = ['$parse'].concat(inject).concat([widget]);
                app
                    .factory(widget, function () { return self })
                    .directive(dName, self[link]);

            };
        }
    }

    //创建控件指令的构造方法
    function initDirective(directive) {
        var
            widget = directive._widgetName,
            //构造link的方法名称
            _widget = "_" + widget;

        addLinker(directive, widget, _widget);
        addInjecter(directive, widget, _widget);

    }

    //创建linker实例成员
    function initMembers(args) {
        var
            x,
            widget = args.base._widgetName;

        for (x in args) {
            //隐藏基类
            if (x != "base") {
                this[x] = args[x];
            }
        }
        this._args = this._parse(this._attrs[widget])(window);
        //表示是否在mapping中设置as
        this._isAs = false;
        this._widgetName = widget;
    }

    //获取绑定上下文_vm
    function convertAs() {
        var
            scope = this._scope,
            args = this._args;
        if (args == null) {
            $$.error('Call "_initMembers" method failed.');
        }
        if (typeof args.as == "string" && !!args.as) {
            this._vm = scope[args.as];
            this._isAs = true;
        }
    }

    //将vm中的方法，属性映射到viewModule(angular to ko)
    function mappingViewModel() {
        var args = this._args;
        for (var j in args.viewModel) {
            var temp = getParent(this, args.viewModel[j]);
            args.viewModel[j] = temp.parent[temp.name];
        }
    }

    //调用控件接口
    function initLinker(args) {
        //指令基类
        var base = args.base;

        initMembers.call(this, args);
        base._initMembers.call(this);

        convertAs.call(this);
        base._convertAs.call(this, transform);

        base._getTemplate.call(this, trimTemplate);

        vistor.call(this, base._widgetName);
        mappingViewModel.call(this);
        base._initWidget.call(this);
        base._addWatcher.call(this);
    }

    //创建指令模块
    function initModule() {
        var module;

        this._uuid = ++uuid;

        if (this._namespace == "aui") {
            //auifw的控件注册到
            this[this._widgetName](angular.module('aui'));
        } else {
            module = this._namespace + '.widgets';
            try {
                angular.module(module);
            } catch (e) {
                angular.module(module, []);
            }
            this[this._widgetName](angular.module(module));
        }
    }

    //创建page类的实例
    function init(name, prototype) {
        var
            createPage,
            fullName,
            statics = _statics,
            _page = page,
            //命名空间
            namespace,
            widget,
            structure = name.split("."),
            instance;

        namespace = structure[0];
        widget = structure[1];
        //挂载到$$directive上的属性名称
        name = (structure + "").replace(/,/g, "_");

        //挂载到body上的属性名称
        fullName = "directive_" + name;
        if ($$directive[name]) {
            //不允许创建同名指令
            $$.error(name + " directive is exist.");
        } else {
            //创建
            instance = _page();
            $$directive[name] = function (name) {
                if (name == "uuid") {
                    return instance._uuid;
                } else if (name == "total") {
                    return uuid + 1;
                }
            };
        }

        //避免指令接口被覆盖
        filter(_directive, prototype);

        $.extend(true, instance, prototype || {});

        instance._namespace = namespace;
        instance._widgetName = widget;

        initDirective(instance);

        initModule.call(instance);
    }

})(jQuery);
```
