# $$page

$$page是一个工厂方法，将页面逻辑对应的JavaScript代码进行封装，并创建一个page类的实例。提供命名空间解析，以及公有(public)、私有（private）方法的区分

- private方法只能在page实例的内部调用。
- public方法可以在page类的不同实例间共享。

```
(function ($) {
    var uuid = -1, body = $('html'), _statics, _reserve;

    if (window.$$page) {
        $.error('$$page is used by other code.Please resolve conflict.');
    };
    /**
     * @param {string} name 命名空间+实例名称
     * #param {object} prototype 原型
     */
    window.$$page = function (name, prototype) {
        init(name, prototype);
    }

    _reserve = ["_controller", "_enum", "_initCallback", "_loaded"];
    _statics = ["$uuid", "$totalPage", "$enum"];

    page.prototype = {
        //在ready函数中调用,在_prepare之后调用
        _create: function () {

        },
        _createPage: function () {
            var self = this;
            self._uuid = ++uuid;
            self._prepare();
            $(function () {
                self._create();
            });

        },
        //定义绑定属性
        _controller: null,
        //定义绑定方法
        _initCallback: $.noop,
        //文档加载完成后触发
        _loaded: $.noop,
        //枚举集合
        _enum: {

        },
        //创建实例时直接调用
        _prepare: function () {

        },
        _trigger: function (e, args, name) {
            var self = e.data.self;

            if ($.isFunction(self._vm[name])) {
                e.data = undefined;
                args.element = undefined;
                self._vm[name].call(self._vm, e, args);
            }
        },
        _apply: function (update) {
            if (!this._scope.$$phase) {
                this._scope.$apply();
            }
        },
        $uuid: function () {
            if (console && console.log) {
                console.log(this._uuid);
            };
        },
        $totalPage: function () {
            if (console && console.log) {
                console.log(uuid + 1);
            };
        },
        /**
         * 获取枚举
         * param {string} type 枚举名称
         */
        $enum: function (type) {
            var value = {};
            if (typeof type == "string") {
                $.extend(true, value, this._enum[type]);
            }
            return value;
        }
    }

    //page类的构造函数
    function page() {
        if (!(this instanceof page)) {
            return new page();
        }
    }

    function transform(as, name) {
        if (name.indexOf(".") == -1 && name != undefined) {
            name = as + '.' + name;
        }

        return name;
    }

    function filter(keys, obj) {
        var i = 0, c = keys.length;

        for (; i < c; i++) {
            delete obj[keys[i]];
        }
    }

    //设置controller实例的默认成员
    function setControllerMember(vm, self, names, args) {
        var i = 0, c = args.length;
        self._vm = vm;

        for (; i < c; i++) {
            self['_' + names[i].replace('$', '')] = args[i];
        }
    }

    function initController(vm, inject, args, fullName) {
        var
            instance = body.data(fullName);

        setControllerMember(vm, instance, inject, args);
        if ($.isFunction(instance._controller)) {
            instance._controller.apply(instance, arguments);
        } else {
            $.error('"_controller" is not implemented.');
        }

        instance._initCallback();
        instance._loaded();

    }

    //创建page类的实例
    function init(name, prototype) {
        var
            createPage,
            fullName,
            statics = _statics,
            _page = page,
            instance;
        //挂载到$$page上的属性名称
        name = (name.split(".") + "").replace(/,/g, "_");
        //挂载到body上的属性名称
        fullName = "page_" + name;
        if ($$page[name]) {
            //合并
            filter(_reserve, prototype);
            $.extend(true, body.data(fullName), prototype);
        } else {
            //创建
            body.data(fullName, _page());
            $$page[name] = function () {
                var
                    options = arguments[0],
                    isMethod = typeof options === "string",
                    args = Array.prototype.slice.call(arguments, 1),
                    instance = body.data(fullName),
                    methodValue;
                if (!instance) {
                    $.error('Cannot call methods on page "' + name + '" prior to initialization; ' +
                        'attempted to call method "' + options + ' "');
                }

                if (isMethod) {
                    if (!$.isFunction(instance[options]) || options.charAt(0) === "_") {
                        $.error('No such method "' + options + '" for "' + name + '" page instance.');
                    }
                    if ($.inArray(options, statics) > -1) {
                        //调用静态方法
                        methodValue = _page.prototype[options].apply(instance, args);
                    } else {
                        methodValue = instance[options].apply(instance, args);
                    }

                    //创建链式调用
                    if (methodValue === undefined || methodValue === instance) {
                        methodValue = $$page;
                    }

                    return methodValue;
                } else {
                    initController(this, $$page[name].$inject, arguments, fullName);
                }
            }
        }

        instance = body.data(fullName);

        $.extend(true, instance, prototype || {});

        page.prototype._createPage.call(instance);
    }
})(jQuery)
```

# hello-combobox.controller.js


```
(function () {
    $$page("helloCombobox.controller", {
        //创建数据上下文(_vm)中的绑定数据
        _controller: function (vm, scope) {
            //可选项集合
            this._vm.colors = [];
            //选中项集合
            this._vm.selectedColorText = {};
        },
        //创建数据上下文(_vm)中的绑定方法。
        _initCallback: function () {
            var self = this;

            self._vm.colorChanged = function (e, args) {
                self._colorChanged(e, args);
            }
        },
        _loaded: function () {
            this._getItems();
        },
        //测试数据
        _getItems: function () {
            var
                self = this,
                i,
                colors = ["#00CCFF", "#3333CC", "#33CC99", "#660066", "#66FF33", "#990000", "#FF9966"];
            for (i = 0; i < 7; i++) {
                self._vm.colors[i] = {
                    color: colors[i]
                };
            }
            //选中第一个元素
            self._vm.selectedColorText = self._vm.colors[0];
        },
        _colorChanged: function (e, args) {
            alert("Selected item is " + this._vm.selectedColorText.color + " .Old selected is " + args.oldValue.item.color);
        }
    });

    //设置Controller
    angular.module('app').controller('HelloComboboxController', $$page.helloCombobox_controller);

    $$page.helloCombobox_controller.$inject = ['$scope'];

})();
```
