<h1>
Container-Components
</h1>

Container Components

There can be quite a bit of code involved in connecting a component to the store.

Putting all this code into the component with heavy rendering logice tends to cause bloated components and violateds the principle of seperation of concerns.

Therefore, it's a common pattern in Redux code to seperate presentaitional components from their connected couterparts, called containers.

<h1>
Comparing presentational and container components
</h1>

The distinction between presentational components and containers is not technical but rather functional.

Presentation components are concerned with how things look and container components are concerned with how things work.

<h1>
Determining where to create containers
</h1>

Not every component needs to be connected to the store.

Generally, you'll only want to create containers for the 'big' components in your app that represent
sections of a page and contain smaller purely functional presentaional components.

These larger container components are responsible for interacting with the store and passing state and dispatch props down to all their presentaional children.

For the fruit stand app, a good starting point would be to create two container components, FruitManagerContainer and FarmerManagerContainer, to respectively render the presentational components for the 'Fruit" and "Farmers" sections of the page.

In general, aim to have fewer containers rather than more.

Most of the components you'll write will be presenetational, but you'll need to generate a few containers to connect presentational components to the Redux store.

<h1>
Writing a container component
</h1>

While you can write a container component from scratch, you can also refactor an existing React component that interacts with a Redux store into separate container and presentational components.

Using a container component to retrieve state

Here's the current version of the FruitList component that subscribes to the store (using store.subscribe) to know when state has been updated and calls store.getState to retrieve and render the fruit state slice:

```js
// ./src/components/FruitList.js

import React from 'react';
import store from '../store';

class FruitList extends React.Component {
  componentDidMount() {
    this.unsubscribe = store.subscribe(() => {
      this.forceUpdate();
    });
  }

  componentWillUnmount() {
    if (this.unsubscribe) {
      this.unsubscribe();
    }
  }

  render() {
    const { fruit } = store.getState();

    return (
      <div>
        {fruit.length > 0
          ? <ul>{fruit.map((fruitName, index) => <li key={index}>{fruitName}</li>)}</ul>
          : <span>No fruit currently in stock!</span>
        }
      </div>
    );
  }
}

export default FruitList;
```

The FruitManager component is responsible for rendering each of the fruit-related components (i.e. FruitList, FruitSeller, FruitQuickAdd, and FruitBulkAdd), so create a container component named FruitManagerContainer to handle all of the store interaction for the "Fruit" section of the page.

As a starting point, here's the code for the FruitManagerContainer component:

```js
    // ./src/components/FruitManagerContainer.js

import React from 'react';
import store from '../store';
import FruitManager from './FruitManager';

class FruitManagerContainer extends React.Component {
  componentDidMount() {
    this.unsubscribe = store.subscribe(() => {
      this.forceUpdate();
    });
  }

  componentWillUnmount() {
    if (this.unsubscribe) {
      this.unsubscribe();
    }
  }

  render() {
    const { fruit } = store.getState();

    return (
      <FruitManager fruit={fruit} />
    );
  }
}

export default FruitManagerContainer;
```

Notice that the container component looks just like the original version.

but instead of directly rendering the fruit state, it sets a prop on the FruitManager component to pass the state down the component hierarchy.

The fruitManager component recieves the fruit porp and in turn uses a prop to pass it down to the FruitList component

```js
    // ./src/components/FruitManager.js

import React from 'react';
import FruitList from './FruitList';
import FruitSeller from './FruitSeller';
import FruitQuickAdd from './FruitQuickAdd';
import FruitBulkAdd from './FruitBulkAdd';

const FruitManager = ({ fruit }) => {
  return (
    <div>
      <h2>Available Fruit</h2>
      <FruitList fruit={fruit} />
      <h2>Fruit Inventory Manager</h2>
      <FruitSeller />
      <FruitQuickAdd />
      <FruitBulkAdd />
    </div>
  );
};

export default FruitManager;
```

And finally, the FruitList component receives the fruit prop and renders it into an unordered list:

```js
    // ./src/components/FruitList.js

import React from 'react';

const FruitList = ({ fruit }) => {
  return (
    <div>
      {fruit.length > 0
        ? <ul>{fruit.map((fruitName, index) => <li key={index}>{fruitName}</li>)}</ul>
        : <span>No fruit currently in stock!</span>
      }
    </div>
  );
};

export default FruitList;
```

Reminder: Using component props to pass a value down the component hierarchy is known as prop threading.

<h1>
Using a container component to dispatch actions
</h1>

Here's the current version of the FruitQuickAdd component that dispatches the ADD_FRUIT action to add a fruit to the fruit stand:

```js
// ./src/components/FruitQuickAdd.js

import React from 'react';
import store from '../store';
import { addFruit } from '../actions/fruitActions';

class FruitQuickAdd extends React.Component {
  addFruitClick = (event) => {
    const fruit = event.target.innerText;
    store.dispatch(addFruit(fruit));
  }

  render() {
    return (
      <div>
        <h3>Quick Add</h3>
        <button onClick={this.addFruitClick}>APPLE</button>
        <button onClick={this.addFruitClick}>ORANGE</button>
      </div>  
    );
  }
}

export default FruitQuickAdd;
```

To prepare to refactor the FruitQuickAdd component into a presentational component, update the FruitManagerContainer component to the following code:

```js
    // ./src/components/FruitManagerContainer.js

import React from 'react';
import store from '../store';
import { addFruit } from '../actions/fruitActions';
import FruitManager from './FruitManager';

class FruitManagerContainer extends React.Component {
  componentDidMount() {
    this.unsubscribe = store.subscribe(() => {
      this.forceUpdate();
    });
  }

  componentWillUnmount() {
    if (this.unsubscribe) {
      this.unsubscribe();
    }
  }

  add = (fruit) => {
    store.dispatch(addFruit(fruit));
  }

  render() {
    const { fruit } = store.getState();

    return (
      <FruitManager
        fruit={fruit}
        add={this.add} />
    );
  }
}

export default FruitManagerContainer;
```

The FruitManager component receives the add prop and in turn uses a prop to pass it down to the FruitQuickAdd component:

```js
    // ./src/components/FruitManager.js

import React from 'react';
import FruitList from './FruitList';
import FruitSeller from './FruitSeller';
import FruitQuickAdd from './FruitQuickAdd';
import FruitBulkAdd from './FruitBulkAdd';

const FruitManager = ({ fruit, add }) => {
  return (
    <div>
      <h2>Available Fruit</h2>
      <FruitList fruit={fruit} />
      <h2>Fruit Inventory Manager</h2>
      <FruitSeller />
      <FruitQuickAdd add={add} />
      <FruitBulkAdd />
    </div>
  );
};

export default FruitManager;
```

And finally, the FruitQuickAdd component receives the add callback function via a prop and calls it within a handleClick event handler, passing in the target button's inner text:

```js
    // ./src/components/FruitQuickAdd.js

import React from 'react';

const FruitQuickAdd = ({ add }) => {
  const handleClick = (event) => add(event.target.innerText);

  return (
    <div>
      <h3>Quick Add</h3>
      <button onClick={handleClick}>APPLE</button>
      <button onClick={handleClick}>ORANGE</button>
    </div>  
  );
};

export default FruitQuickAdd;
```

The change between the original and refactored FruitQuickAdd component isn't as dramatic as the FruitList component example, but it's still a significant improvement to the overall separation of concerns. The FruitQuickAdd component is now strictly concerned with rendering the UI and handling user generated events (i.e. button clicks) and the FruitManagerContainer component is now strictly concerned with interacting with the Redux store.

<h1>
Reviewing the completed container component
</h1>

The FruitManagerContainer container component can continue to be expanded until each of its child presentational components no longer interact directly with the store. Here's a look at the completed FruitManagerContainer component:

```js
    // ./src/components/FruitManagerContainer.js

import React from 'react';
import store from '../store';
import { addFruit, addFruits, sellFruit, sellOut } from '../actions/fruitActions';
import FruitManager from './FruitManager';

class FruitManagerContainer extends React.Component {
  componentDidMount() {
    this.unsubscribe = store.subscribe(() => {
      this.forceUpdate();
    });
  }

  componentWillUnmount() {
    if (this.unsubscribe) {
      this.unsubscribe();
    }
  }

  add = (fruit) => {
    store.dispatch(addFruit(fruit));
  }

  addBulk = (fruit) => {
    store.dispatch(addFruits(fruit));
  }

  sell = (fruit) => {
    store.dispatch(sellFruit(fruit));
  }

  sellAll = () => {
    store.dispatch(sellOut());
  }

  render() {
    const { fruit } = store.getState();
    const distinctFruit = Array.from(new Set(fruit)).sort();

    return (
      <FruitManager
        fruit={fruit}
        distinctFruit={distinctFruit}
        add={this.add}
        addBulk={this.addBulk}
        sell={this.sell}
        sellAll={this.sellAll} />
    );
  }
}

export default FruitManagerContainer;
```