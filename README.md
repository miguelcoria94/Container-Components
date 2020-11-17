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



