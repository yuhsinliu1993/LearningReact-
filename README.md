# Learning React 

## Component

Components let you split the UI into independent, reusable pieces, and think about each piece in isolation.

 ```JS
 import React from 'react'

 class ContractInput extends React.Component {

     # must have render() function     
     render() {
         return (
             ...
         )
     }
 }
 ```

* Overview
    * `React.Component` is an abstract base class, so it rarely makes sense to refer to
    * `React.Component` directly. Instead, you will typically subclass it, and define at least a render() method.

    ```JS  
    # This is for ES6
    class Greeting extends React.Component {
      render() {
        return <h1>Hello, {this.props.name}</h1>;
      }
    }
    ```

* Life Cycle

    ![](pics/React_cycle_life.png)

    Each component has several “lifecycle methods” that you can override to run code at particular times in the process.

    Methods prefixed with`will` are called right before something happens.

    Methods prefixed with `did` are called right after something happens.

* Mounting
These methods are called when an instance of a component is being created and inserted into the DOM:

    1. [constructor()](https://reactjs.org/docs/react-component.html#constructor)
    2. [static getDerivedStateFromProps()](https://reactjs.org/docs/react-component.html#static-getderivedstatefromprops)
    3. [componentWillMount() / UNSAFE_componentWillMount()](https://reactjs.org/docs/react-component.html#unsafe_componentwillmount)
    4. [render()](https://reactjs.org/docs/react-component.html#render)
    5. [componentDidMount()](https://reactjs.org/docs/react-component.html#componentdidmount)     




在React中，有提供兩個屬性進行資料的傳遞與儲存－`this.props` 以及 `this.state`。
這兩個屬性有不同的特性與行為，在使用上，大概可以區分為靜態資料使用`this.props`，動態資料使用`this.state`。
另外還有一個屬性`this.refs`，是用來取得UI元素的參考用的。

1. What is `this.props` ?

    this.props通常是使用在父元素傳值給子元素的情境上，直接看Example比較快。

    ```JS
    var App = React.createClass({
  		render: function() {
    		return <div>Hello~  {this.props.name}</div>;
  		}
	});

    React.render(<App name="foo"/>, document.getElementById('example'));

    # Output: Hello~ foo
    ```

    基本上，當父元素想傳值給子元素時，會設定在標籤的Attribute上。子元素就能夠透過`this.props.Attribute`以取得父元素傳進來的值。在這Example中，我們透過`name="foo"`傳給子元素

    若需要由子元素傳資料到父元素時，該怎麼辦呢？在React.JS的官方Sample中，有這麼一個處理方式。就是由父元素設定一個function到一個Attribute上，子元素再把要傳給父元素的資料放到function的arguments。這樣父元素的function就可以取得子元素傳進來的資料了。

    ```JS
    var App =  React.createClass({
      handleChildCallback: function(childValueObj){   
    console.log(childValueObj);
    this.setState({data:childValueObj.value});
      },
      getInitialState: function(){
        return {data:[]};
      },
      render: function() {
        return (
          <div>
            <h2>Hello {this.state.data}</h2>
            <AppChild ParentsFunction={this.handleChildCallback}/>
          </div>
        );
      }
    });

    var AppChild = React.createClass({
      onChangeHandler: function(){
        var childValue = React.findDOMNode(this.refs.text).value;    
        this.props.ParentsFunction({value:childValue});    
      },
      render: function(){
        return (      
            <input type="text" onChange={this.onChangeHandler} ref="text"/>      
         );      
      }
    });

    React.render(<App />, document.getElementById('example'));
    ```
