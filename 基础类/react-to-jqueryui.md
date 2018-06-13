# 中转React组件和jQueryUI控件

```
import React, { Component } from 'react';
import ReactDOM from 'react-dom';

var $ = global.jQuery;

var uuid = -1;

var $$ =  global.$$;

class DependencyObject extends Component {
    uuid = ++uuid;
    /**
     * 是否禁止组件更新。
     * {default} true，表示不禁止
     */
    shouldUpdate = true;

    /**
     * tool tip
     */
    tooltip = "";

    componentDidMount() {
        this.element = $(ReactDOM.findDOMNode(this));

        if (typeof this.props.tooltip === "string" && this.props.tooltip !== "") {
            this.element.attr('title', this.props.tooltip);
        }

        if (typeof this.element.attr('data-part') !== 'string') {
            let part = this.props['data-part'];
            if (typeof part !== 'string') {
                part = 'none';
            }
            this.element.attr('data-part', part);
        }
    }

    setHandlersBind(names) {
        let c;

        if ($.isArray(names)) {
            c = names.length;

            for (let i = 0; i < c; i++) {
                if ($.isFunction(this[names[i]])) {
                    this[names[i]] = this[names[i]].bind(this);
                } else {
                    $.error(names[i]+' is not a function.');
                }
            }
        }

        return this;
    }

    componentWillUnmount() {
        //console.log('DependencyObject destroy')
    }

    shouldComponentUpdate() {
        return this.shouldUpdate;
    }

    /**
     * 触发行自定义事件
     * {string} callbackName 回调函数的名称
     * {jQuery.Event | React.Event} e 原生事件参数（可能被jQuery封装或react封装）
     * {$$.Event} args 额外参数
     */
    trigger(callbackName, e, args) {
        if ($.isFunction(this.props[callbackName])) {
            this.props[callbackName](e, args);
        }
    }
}

class UIElement extends DependencyObject {
    constructor(props) {
        super(props);
        this.shouldUpdate = false;

        this.setHandlersBind([
            'setShouldUpdate',
        ]);
    }

    /**
     * 在触发componentDidMount回调是，调用该接口。在该接口中，初始化控件实例，子类需要实现
     */
    _init() {
        //$$.log(this.widgetName + " does not implement interface member '_init'.");
    }

    /**
       * 控件销毁时调用该接口，需要特殊销毁逻辑的子类实现该接口
       */
    _destroy() {
        //$$.log(this.widgetName+" does not implement interface member '_destroy'.")
    }

    componentDidMount() {
        super.componentDidMount();

        this.element[0].setShouldUpdate = this.setShouldUpdate;

        if ($.isFunction(this.props.widgetRef)) {
            this.props.widgetRef(this.bridge());
        }

        if (typeof this.props.id === "string" && this.props.id.length > 0) {
            this.element.attr('id', this.props.id)
        }

        this._init();

        //控件初始化完成之后触发loaded事件
        if ($.isFunction(this.props.loaded)) {
            this.props.loaded.call(this);
        }
    }

    setShouldUpdate(shouldUpdate) {
        this.shouldUpdate = shouldUpdate;
    }

    widget(name, value){
        return this.element[this.widgetName](name, value);
    }

    bridge(name, value) {
        var self = this;
        return (...args) => {
            self.element[self.widgetName].apply(self.element,args);
        };
    }

    componentWillReceiveProps(nextProps,fourceUpdateAssert) {
        var
            widget,
            ele = this.element,
            prop;

        if(!$.isFunction(fourceUpdateAssert)){
            fourceUpdateAssert = function(){
                return false;
            }
        }

        if (ele === undefined) {
            $.error("Init '" + this.widgetName + "' widget failed.Please check your code.");
        }

        widget = this.element[this.widgetName];

        for (prop in nextProps) {
            let newValue = nextProps[prop];
            //值比较
            if (this.props[prop] != newValue || fourceUpdateAssert(prop)) {
                widget.call(ele, "option", prop, newValue);
            }
        }
    }

    componentWillUnmount() {
        try {
            this.element[this.widgetName]("destroy");
            this._destroy();
            if ($.isFunction(this.element[0].setShouldUpdate)) {
                this.element[0].setShouldUpdate(false);
                delete this.element[0].setShouldUpdate;
            }
        } catch (e) {
            //$$.log(this.widgetName + " destroy error: " + e.message);
        }
    }

    //解除组件刷新限制
    forceRender() {
        this.shouldUpdate = true;
    }

    shouldComponentUpdate() {
        return this.shouldUpdate;
    }

    /**
     * 触发行自定义事件
     * {jQuery.Event | React.Event} e 原生事件参数（可能被jQuery封装或react封装）
     * {$$.Event} args 额外参数
     * {function} callback 回调函数
     */
    trigger(e, args, callback) {
        if ($.isFunction(callback)) {
            e.data = undefined;
            args && (args.element = undefined);

            callback.apply(this, [e, args]);
        }
    }

    render() {
        return (<div>{this.props.children}</div>);
    }
}

class PageComponent extends DependencyObject {
    constructor(props) {
        super(props);
        this.data = {};
        this.componentInit();
    }

    componentInit(){

    }

    trigger(e, args) {
        let data;
        if (args instanceof $$.Event) {
            data = args;
        } else {
            data = $$.Event({
                oldValue: {},
                newValue: {
                },
                parameters: args
            });
        }
        super.trigger('dataChanged', e, data);
    }
}

export { DependencyObject,UIElement, PageComponent}
```
