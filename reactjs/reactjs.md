# Reactjs notes

- [Reactjs notes](#reactjs-notes)
  - [architecure](#architecure)
    - [containers & presentation components](#containers--presentation-components)
    - [resource component](#resource-component)
      - [example](#example)
  - [context](#context)
    - [Global State component](#global-state-component)
  - [hooks](#hooks)
    - [custom hooks](#custom-hooks)
      - [http](#http)
      - [useHttp](#usehttp)
    - [shouldComponentUpdate replacement](#shouldcomponentupdate-replacement)
      - [examples](#examples)
    - [useEffect](#useeffect)
      - [examples](#examples-1)
        - [traditional state replica](#traditional-state-replica)
          - [updating state](#updating-state)
    - [useMemo](#usememo)
    - [useReducer](#usereducer)
    - [useState](#usestate)
      - [examples](#examples-2)
        - [component did mount + did mount + will mount](#component-did-mount--did-mount--will-mount)
          - [updating state](#updating-state-1)

## architecure

### containers & presentation components

- ExampleContainer & ExampleWidget

### resource component

```javascript
class Resource extends Component {

  state = {
    loading: false,
    payload: null
  };

  componentDidMount() {
    this.setState({loading: true});
     axios.get(this.props.path).then(res => {
       this.setState({
         loading: false,
         payload: res.data
       });
     });
  }

  render() {
    return this.props.render(this.state);
  }
}

```

#### example

```jsx
<Resource
path="/api/puppies"
render={data => {
  if(data.loading)  
  {
    return <p>Loading...</p>
  }  
  else  
  {
    return data.payload.map(item => <div>{item}</div>)
  }
}}
/>
```

---

## context

**set-up**

```javascript
// context/shop.context.js
import React, { createContext } from 'react';

export default createContext(
  {
    products: [
      { id: 'p1', title: 'Gaming Mouse', price: 29.99 },
      { id: 'p2', title: 'Harry Potter 3', price: 9.99 },
      { id: 'p3', title: 'Used plastic bottle', price: 0.99 },
      { id: 'p4', title: 'Half-dried plant', price: 2.99 }
    ],
    cart: [],
    addProductToCart: (product) => {},
  removeProductToCart: (productId) => {}
  }, []
);
```

**provider**

```javascript
// context/shop.context.js
import ShopContext from './context/shop-context';
// ...
  return (
      <ShopContext.Provider value={{
        products: this.state.products,
        cart: this.state.cart,
        addProductToCart: this.addProductToCart,
        removeProductToCart: this.removeProductToCart
      }}>
        <BrowserRouter>
          <Switch>
            <Route path='/' component={ProductsPage} exact />
            <Route path='/cart' component={CartPage} exact />
          </Switch>
        </BrowserRouter>
      </ShopContext.Provider>
    );
// ...
```

**consumer**

**first way**

```javascript
//  ####################
//  Adv: 
//  - Works in both class and functional components
//  Dis:
//  - Context is scoped to Render()
//  ####################
// context/shop.context.js
import ShopContext from './context/shop-context';
// ...
  return (
      <ShopContext.Consumer>
        {context => (
          <React.Fragment>
            <main className='products'>
              <ul>
                {context.products.map(product => (
                  <li key={product.id}>
                    <div>
                      <strong>{product.title}</strong> - ${product.price}
                    </div>
                    <div>
                      <button
                        onClick={context.addProductToCart.bind(this, product)}
                      >
                        Add to Cart
                      </button>
                    </div>
                  </li>
                ))}
              </ul>
            </main>
          </React.Fragment>
        )}
      </ShopContext.Consumer>
    );
// ...
```
**consumer**

**second way**

```javascript
//  ####################
//  Adv: 
//  - Works with entire class object not constrained to render function
//  Dis:
//  - Class only
//  ####################
// context/shop.context.js
import ShopContext from './context/shop-context';
class CartPage extends Component {
  static contextType = ShopContext;

  componentDidMount() {
    console.log(this.context);
  }
  render() {
    return (
      <React.Fragment>
        <main className='cart'>
          {this.context.cart.length <= 0 && <p>No Item in the Cart!</p>}
          <ul>
            {this.context.cart.map(cartItem => (
              <li key={cartItem.id}>
                <div>
                  <strong>{cartItem.title}</strong> - ${cartItem.price} (
                  {cartItem.quantity})
                </div>
                <div>
                  <button
                    onClick={this.context.removeProductFromCart.bind(
                      this,
                      cartItem.id
                    )}
                  >
                    Remove from Cart
                  </button>
                </div>
              </li>
            ))}
          </ul>
        </main>
      </React.Fragment>
    );
  }
}
// ...
```

### Global State component

```javascript
// context/GlobalState.js
// contains context and renders children

import React, { Component } from 'react';
import ShopContext from './shop-context';

export default class GlobalState extends Component {
  state = {
    products: [
      { id: 'p1', title: 'Gaming Mouse', price: 29.99 },
      { id: 'p2', title: 'Harry Potter 3', price: 9.99 },
      { id: 'p3', title: 'Used plastic bottle', price: 0.99 },
      { id: 'p4', title: 'Half-dried plant', price: 2.99 }
    ],
    cart: []
  };

  addProductToCart = product => {
    const updatedCart = [...this.state.cart];
    const updatedItemIndex = updatedCart.findIndex(
      item => item.id === product.id
    );

    if (updatedItemIndex < 0) {
      updatedCart.push({ ...product, quantity: 1 });
    } else {
      const updatedItem = {
        ...updatedCart[updatedItemIndex]
      };
      updatedItem.quantity++;
      updatedCart[updatedItemIndex] = updatedItem;
    }

    // async
    setTimeout(() => {
      console.log('api request');
    }, 3000);

    this.setState({ cart: updatedCart });
  };

  removeProductFromCart = productId => {
    const updatedCart = [...this.state.cart];
    const updatedItemIndex = updatedCart.findIndex(
      item => item.id === productId
    );

    const updatedItem = {
      ...updatedCart[updatedItemIndex]
    };
    updatedItem.quantity--;
    if (updatedItem.quantity <= 0) {
      updatedCart.splice(updatedItemIndex, 1);
    } else {
      updatedCart[updatedItemIndex] = updatedItem;
    }

    // async
    setTimeout(() => {
      console.log('api request');
    }, 3000);

    this.setState({ cart: updatedCart });
  };

  render() {
    return (
      <ShopContext.Provider
        value={{
          products: this.state.products,
          cart: this.state.cart,
          addProductToCart: this.addProductToCart,
          removeProductFromCart: this.removeProductFromCart
        }}
      >
        {this.props.children}
      </ShopContext.Provider>
    );
  }
}

```

**usage**

```javascript
// ...
import GlobalState from './context/GlobalState';

class App extends Component {
  render() {
    return (
      <GlobalState>
        <BrowserRouter>
          <Switch>
            <Route path='/' component={ProductsPage} exact />
            <Route path='/cart' component={CartPage} exact />
          </Switch>
        </BrowserRouter>
      </GlobalState>
    );
  }
}

export default App;
```

---

## hooks

### custom hooks

#### http
```javascript
import { useState, useEffect } from 'react';

export const useHttp = (url, dependencies) => {
  const [isLoading, setIsLoading] = useState(false);
  const [fetchData, setFetchData] = useState(null);

  useEffect(() => {
    setIsLoading(true);
    
    fetch(url)
      .then(response => {
        if (!response.ok) {
          throw new Error('Could not fetch person!');
        }
        return response.json();
      })
      .then(data => {
        setFetchData(data);
        setIsLoading(false);
      })
      .catch(err => {
        console.log(err);
      });
  });

  return [isLoading, fetchData];
};
```

#### useHttp
```javascript
const [isLoading, fetchData] = useHttp('https://swapi.co/api/people', []);
```

### shouldComponentUpdate replacement

#### examples

```javascript
export default React.memo(CharPicker);
```

```javascript
export default React.memo(CharPicker, (prevProps, nextProps) => {
  // return true if props are equal
  // dont rerender
  return (
      nextProps.selectedChar === prevProps.selectedChar
    );

  // return false 
  // should rerender

});
```

### useEffect

#### examples

##### traditional state replica

```javascript
const [state, setState] = useState({
    selectedCharacter: 1,
    side: 'light',
    destroyed: false
  });
```

###### updating state

```javascript
setState({ ...state, side: side });
```

### useMemo

```javascript
// parent
import React from "react";
import ReactDOM from "react-dom";
import { Count } from "./Count";

import "./styles.css";

function App() {
  const [text, setText] = React.useState("");
  const [text2, setText2] = React.useState("");

  const onOdd = React.useCallback(() => setText(""), [setText]);
  const data = React.useMemo(
    () => ({
      text2,
      isEven: text2.length % 2 === 0
    }),
    [text2]
  );

  return (
    <div className="App">
      <input value={text} onChange={e => setText(e.target.value)} />
      <input
        placeholder="text2"
        value={text2}
        onChange={e => setText2(e.target.value)}
      />
      <Count onOdd={onOdd} data={data} />
    </div>
  );
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```
```javascript
// child

import React from "react";

export const Count = React.memo(({ onOdd }) => {
  const [count, setCount] = React.useState(0);
  const renders = React.useRef(0);
  return (
    <div className="App">
      <div>count: {count}</div>
      <div>renders: {renders.current++}</div>
      <button
        onClick={() => {
          if (count % 2 === 0) {
            onOdd();
          }
          setCount(c => c + 1);
        }}
      >
        increment
      </button>
    </div>
  );
}
// , (prevProps, nextProps) => {
//   if(prevProps.data.isEven !=== nextProps.data.isEven) {
//     return false
//   } else {
//     return true
//   }
});

```
### useReducer
- reducers file @  context/reducers
  
```javascript

```

### useState 

#### examples 

##### component did mount + did mount + will mount

- Array of dependency values
- Empty dependencies resembles `componentDidMount` lifecyle hook
  - with a return resembles `componentWillUnmount` lifecyle hook because useEffect will run only when component is mounted and the return fires when the components unmounts
- return function executes before each subsequent useEffect call
  
```javascript
  useEffect(() => {
    fetchData();

    return () => {
      console.log('clean up...');
    };
  }, [props.selectedChar]);
```

###### updating state

```javascript
const [chosenSide, setChosenSide] = useState('light');

  const [selectedCharacter, setselectedCharacter] = useState(1);

  const [destroyed, setDestroyed] = useState(false);
```