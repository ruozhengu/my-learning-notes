

# Redux & React

> Redux is a predictable state container for JavaScript apps.
>
> The whole goal of Redux is to increase predictability



### Lesson 1: Managing State:

- Main goal of Redux is to make the state management of an application more predictable. 

-  State Tree

  > One of the key points of Redux is that all of the data is stored in a single object called the *state tree*	

  ```js
  {
    recipes: [
      { … },
      { … },
      { … }
    ],
    ingredients: [
      { … },
      { … },
      { … },
      { … },
      { … },
      { … }
    ],
    products: [
      { … },
      { … },
      { … },
      { … }
    ]
  }
  ```

  Like above code, all states all stored in one location, in single object. This location is called state tree.

  Three ways to interact with it:

  * Get the state
  * listening for state changes
  * update the state

  `The store = state tree + getting the state + listening for changes + updating state`

  

- Code to create `Store`, then have `Store` keep track of the state

  1. Open a `index.js` file and build a function to create a new store

  2. create state variable to hold state for entire application. Then provide access to it.

  3. returns an object that publicly exposes the `getState` function

  4. Create a subscriber method, user can then pass it a callback function that whenever a state changes internally, we can invoke this callback function and user can do whatever they want.

     So now, we can listen for changes in the state.

  5. Next, lets update the state. Note that **Only an event can change the state of the store.**

     > When an event takes place in a Redux application, we use a plain JavaScript object to keep track of what the specific event was. This object is called an **Action**.
     >
     > Example of Action:

     ```js
     {
       type: "ADD_PRODUCT_TO_CART",
       productId: 17
     }
     ```

     >  What makes this plain JavaScript object special in Redux, is that *every Action must have a `type` property*. The purpose of the `type` property is to let our app (Redux) know *exactly* what event just took place. This Action tells us that a product was added to the cart.

     > With ProductId, we know exactly which product is added each time.

     **Action Creators** are functions that create/return action objects. For example: 

     ```js
     const addItem = item => ({
       type: ADD_ITEM,
       item
     });
     ```

  6. Now that we have both actions and state tree, lets finally update the state. We write a function takes current state and action, then based on action, function will return new updated state. This function needs to be predictable. We should predict the return value based on input. We call this type of function **Pure Function**

     **The function that returns the new state needs to be a pure function.**

     1. Return the same result if the same arguments are passed in
     2. Depend solely on the arguments passed into them [no access to outside scope]
     3. Do not produce side effects, such as API and I/O operations [no interaction with outside]

  ```js
  //LIBRARY CODE --------------------
  
  function createStore(reducer) {
    // The store should have four parts
    // 1. the state
    // 2. get the state
    // 3. listen to changes on the state
    // 4. update the state
    
    let state //hold state of entire app
    let listeners // hold all subscribed listeners
    
    const getState = () => state //just to return state of entire app
    
    const subscribe = (listener) => {
      listener.push(listener); //when state change, all subscriber will invoke
      
      // provide a function so user can remove subscribtion
      return function() => { //return a function that user can unsubscirbe 
        listeners = listeners.filter(function(l) {
         l !== listener 
        })
      };
    };
    
    //update the actual state inside of Store
    const dispatch = (action) -> { //takes action which will change state
      state = reducer(state, action) //inspect action and actually change state
      listeners.forEach((listener) => listener()) //invoke each function as state changes
    }
    
    return {
      getState, //expose this so that user can get app's state
      subscribe,
      dispatch
    }
}
  ```
  
  ```js
  // APP CODE --------------------
  
  //A Reducer Function | pure function to return new state
  function todos (state = [], action) { //state = []: if state is undefined, it is []
    if (action.type === "ADD_TODO") {
      return state.concat([action.todo]) //concat returns a new array, so still pure
    } else {
      return state
    }
  }
  
  // important: create store
  const store = createStore(todos) //pass reducer function 
  
  // subscirbe listeners which is a console log function
  store.subscribe(() => {//call subscribe multiple times
    console.log('The new state is ', store.getState())
  })
  
  // change state to add sth, trigger listener
  store.dispatch({ //pass in action
    type: "ADD_TODO",
    todo: {
      id: 0,
      name: 'learn redux',
      complete: false
    }
  })
  
  // how to unsubscirbe: first call subscirbe and get returned function
const unsubscribe = store.subscribe(() => {
    console.log('The store changed')
})
  
  // unsubscribe now
  unsubscribe()
  ```

  Note that when `store.subscribe()` is called, the function passed into it is not invoked!

  **- Remember that our two rules are :**

  1. Only an event can change the state of the store.
2. The function that returns the new state needs to be a pure function.

**- We've finally finished creating the `createStore` function! Using the image above as a guide, let's br eak down what we've accomplished:**

- we created a function called `createStore()` that returns a *store* object
  
- `createStore()` must be passed a "reducer" function when invoked
  
- the store object has three methods on it:
  
    - `.getState()` - used to get the current state from the store
    
    - `.subscribe()` - used to provide a listener function the store will call when the state changes
    
    - `.dispatch()` - used to make changes to the store's state
    
      

* Next, lets manage more states/actions than just adding todo. Such as `REMOVE_TODO` and `TOGGLE_TODO`

  1. change our reducer function to handle more actions:

     ```js
     //A Reducer Function 
     function todos (state = [], action) {
       if (action.type === "ADD_TODO") {
         return state.concat([action.todo]) //concat returns a new array, so still pure
       } else if (action.type === "REMOVE_TODO") {
         return state.filter((todo) => todo.id !== action.id)
       } else if (action.type === "TOGGLE_TODO") { // you cannot just mutate in pure function. Need to use object.assign to create new object with merged properties
         return state.map((todo) => todo.id !== action.id ? todo :
                         Object.assign({}, todo, {complete: !todo.complete}))
       } else {
         return state
       }
     }
     ```

  2. Lets add functionality to also takes in `goals` rather than `todos` in our state.

  ```js
  function goals (state= [], action) {
    switch(action.type) {
      case 'ADD_GOAL' :
        return state.concat([action.goal])
      case 'REMOVE_GOAL' :
        return state.filter((goal) => goal.id !== action.id)
      default :
        return state
    }
  }
  ```

  ==However, the `createStore()` function we built can only handle a *single* reducer function,As your app grows more complex, you'll want to split your reducing functioninto separate functions, each managing independent parts of the state.==

  We can create a ==root reducer== that decides which reducer to pass in.

  <img src="/Users/gabriel/Desktop/Screen Shot 2020-01-30 at 3.58.03 PM.png" alt="Screen Shot 2020-01-30 at 3.58.03 PM" style="zoom:33%;" />

  ```js
  function app (state = {}, action) {
    return {
      todos: todos(state.todos, action),
      goals: goals(state.goals, action),
    }
  }
  
  /*
  Passing the root reducer to our store since our createStore function can only take one reducer.
  */
  
  const store = createStore(app);
  ```

* Better Practice: write a function to create actions rather than manually write so many actions for dispatch

  ```js
  const ADD_TODO = 'ADD_TODO'
  function addTodoAction(todo) {
    return {
      type: ADD_TODO,
      todo,
    }
  }
  store.dispatch(addTodoAction({id: 1, name: 'wash clothes', complete: false})
  ```

  


### Lession 2: UI + Redux

1. keep in mind the previous code we write. 
2. We will create a new `index.js` to add some UIs.
3. Next we will add todo list area and goals area like:

<img src="/Users/gabriel/Library/Application Support/typora-user-images/image-20200316224609078.png" alt="image-20200316224609078" style="zoom: 67%;margin-left:0" />

4. `<script src="https://cdnjs.cloudflare.com/ajax/libs/redux/3.7.2/redux.min.js"></script>`Download real redux and stick it to the global namespace as `Redux` With this we can delete all the library code we have before.

```html
<!DOCTYPE html>
<html>
<head>
  <title>Udacity Todos Goals</title><script src="https://cdnjs.cloudflare.com/ajax/libs/redux/3.7.2/redux.min.js"</script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/redux/3.7.2/redux.min.js"</script>
</head>
<body>
	
  <script> 
    // App Code
    const ADD_TODO = 'ADD_TODO'
    const REMOVE_TODO = 'REMOVE_TODO'
    const TOGGLE_TODO = 'TOGGLE_TODO'
    const ADD_GOAL = 'ADD_GOAL'
    const REMOVE_GOAL = 'REMOVE_GOAL'

    function addTodoAction (todo) {
      return {
        type: ADD_TODO,
        todo,
      }
    }

    function removeTodoAction (id) {
      return {
        type: REMOVE_TODO,
        id,
      }
    }

    function toggleTodoAction (id) {
      return {
        type: TOGGLE_TODO,
        id,
      }
    }

    function addGoalAction (goal) {
      return {
        type: ADD_GOAL,
        goal,
      }
    }

    function removeGoalAction (id) {
      return {
        type: REMOVE_GOAL,
        id,
      }
    }

    function todos (state = [], action) {
      switch(action.type) {
        case ADD_TODO :
          return state.concat([action.todo])
        case REMOVE_TODO :
          return state.filter((todo) => todo.id !== action.id)
        case TOGGLE_TODO :
          return state.map((todo) => todo.id !== action.id ? todo :
            Object.assign({}, todo, { complete: !todo.complete }))
        default :
          return state
      }
    }

    function goals (state = [], action) {
      switch(action.type) {
        case ADD_GOAL :
          return state.concat([action.goal])
        case REMOVE_GOAL :
          return state.filter((goal) => goal.id !== action.id)
        default :
          return state
      }
    }

    //function app (state = {}, action) {
      //return {
        //todos: todos(state.todos, action),
        //goals: goals(state.goals, action),
      //}
    //}
    
    // use combineReducers to combine reducer functions
    const store = Redux.createStore(Redux.combineReducers({
      todos,
      goals,
    }))
    
    store.subscribe(() => {
      const { goals, todos } = store.getState()

      document.getElementById('goals').innerHTML = ''
      document.getElementById('todos').innerHTML = ''

      goals.forEach(addGoalToDOM)
      todos.forEach(addTodoToDOM)
    })

    // DOM code
     function addTodoToDOM (todo) {
      const node = document.createElement('li')
      const text = document.createTextNode(todo.name)

      const removeBtn = createRemoveButton(() => {
        store.dispatch(removeTodoAction(todo.id))
      })

      node.appendChild(text)
      node.appendChild(removeBtn)
      node.style.textDecoration = todo.complete ? 'line-through' : 'none'
      node.addEventListener('click', () => {
        store.dispatch(toggleTodoAction(todo.id)) //if click, state changes, call dispatch
      })

      document.getElementById('todos')
        .appendChild(node)
    }

    function addGoalToDOM (goal) {
      const node = document.createElement('li')
      const text = document.createTextNode(goal.name)
      const removeBtn = createRemoveButton(() => {
        store.dispatch(removeGoalAction(goal.id))
      })

      node.appendChild(text)
      node.appendChild(removeBtn)

      document.getElementById('goals')
        .append(node)
    }
    
    
    function addTodo () { //call dispatch func
      const input = document.getElementById('todo')
      const name = input.value
      input.value = ''

      store.dispatch(addTodoAction({
        name,
        complete: false,
        id: generateId()
      }))
    }

    function addGoal () { //call dispatch func
      const input = document.getElementById('goal')
      const name = input.value
      input.value = ""
      
      store.dispatch(addGoalAction({
        id: generateId(),
        name,
      }))
    }

    document.getElementById('todoBtn')
      .addEventListener('click', addTodo) //make button clickable

    document.getElementById('goalBtn')
      .addEventListener('click', addGoal) //make button clickable

    function createRemoveButton (onClick) {
      const removeBtn = document.createElement('button')
      removeBtn.innerHTML = 'X'
      removeBtn.addEventListener('click', onClick)
      return removeBtn
    }
  </script>
</body>
  <div>
    <h1>Todo List</h1>
    <input id='todo' type='text' placeholder='Add Todo' />
    <button id='todoBtn'>Add Todo</button>
    <ul id='todos'></ul>
  </div>
  <div>
    <h1>Goals</h1>
    <input id='goal' type='text' placeholder='Add Goal' />
    <button id='todoBtn'>Add Todo</button>
    <ul id='goals'></ul>
  </div>
</html>
```

**Redux Composition**

Lets see how Redux can help manage not only each section of Redux store but also any nested data as below. Our goal is that user might change avatar and we will handle it (tweets->[user]->author->avatar)

```json
{
  users: {},
  setting: {},
  tweets: {
    btyxlj: {
      id: 'btyxlj',
      text: 'What is a jQuery?',
      author: {
        name: 'Tyler McGinnis',
        id: 'tylermcginnis',
        avatar: 'twt.com/tm.png'
      }   
    }
    btyxlj2: {
      id: 'btyxlj2',
      text: 'What is a jQuery2?',
      author: {
        name: 'Tyler McGinnis2',
        id: 'tylermcginnis2',
        avatar: 'twt.com/tm.png2'
      }   
    }
  }  
}
```

We first need to create a root reducer using `combineReducers` method:

```js
const reducer = combineReducers({
  users,
  settings,
  tweets
})
```

`combineReducers` is responsible for invoking all the other reducers, passing them the portion of their state that they care about.

Since avatar is under "tweets", that's the state data that we care now, let's create a tweet reducer.

```js
function tweets (state = {}, action) {
  switch(action.type){
      case ADD_TWEET :
        ...
      case REMOVE_TWEET :
        ...
      case UPDATE_AVATAR :
        return {
          ...state,
          [action.tweetId]: {
            ...state[action.tweetId],
            author: {
              ...state[action.tweetId].author,
              avatar: action.newAvatar 
            }
          }
        }
  }
}
```

Because of immutability requirement, we need lots of spread operator to copy information. To have good coding style, write the followings to have helper function handles subcomponents:

```js
function author (state, action) {
  switch (action.type) {
      case : UPDATE_AVATAR
        return {
          ...state,
          avatar: action.newAvatar
        }
      default :
        state
  }
}

function tweet (state, action) {
  switch (action.type) {
      case ADD_TWEET :
        ...
      case REMOVE_TWEET :
        ...
      case : UPDATE_AVATAR
        return {
          ...state,
          author: author(state.author, action)
        }
      default :
        state
  }
}

function tweets (state = {}, action) {
  switch(action.type){
      case ADD_TWEET :
        ...
      case REMOVE_TWEET :
        ...
      case UPDATE_AVATAR :
        return {
          ...state,
          [action.tweetId]: tweet(state[action.tweetId], action)
        }
      default :
        state
  }
```

You can control state key names by using different keys for the reducers in the passed object. For example, you may call `combineReducers({ todos: myTodosReducer, counter: myCounterReducer })` for the state shape to be `{ todos, counter }`.

A popular convention is to name reducers after the state slices they manage, so you can use ES6 property shorthand notation: `combineReducers({ counter, todos })`. This is equivalent to writing `combineReducers({ counter: counter, todos: todos })`.

#### MiddleWare

Example: back to our todo list web, now whenever the user add a todo that contains word "bitcoin", gives an alert. 

To do this, we need to hook into the moment after dispatch function is triggered but before it hits the reducer and actually modifies state.

```js
function checkAndDispatch (store, action) {
      if (
        action.type === ADD_TODO &&
        action.todo.name.toLowerCase().includes('bitcoin')
      ) {
        return alert("Nope. That's a bad idea.")
      }

      if (
        action.type === ADD_GOAL &&
        action.goal.name.toLowerCase().includes('bitcoin')
      ) {
        return alert("Nope. That's a bad idea.")
      }

      return store.dispatch(action)
}
```

We would like to hijack the given dispatch function from `react-redux` and also change all code with `dispatch`keyword to include a new argument `store`

```js
checkAndDispatch(store, addTodoAction({
        name,
        complete: false,
        id: generateId()
}))
```



> We would like to have checkAndDispatch function inside the dispatch function and the actual reducer call, this is called middleware.

Between the dispatching of an action and the reducer running, we can introduce code called **middleware** to *intercept* the action before the reducer is invoked

What's great about middleware is that once it receives the action, it can carry out a number of operations, including:

- producing a side effect (e.g., logging information about the store)
- processing the action itself (e.g., making an asynchronous HTTP request)
- redirecting the action (e.g., to another piece of middleware)
- dispatching supplementary actions

 

Below are the changes to the code:

1. add below function. next is either the next middleware if exist or the dispatch function.

```js
    const checker = (store) => (next) => (action) => {
      if (
        action.type === ADD_TODO &&
        action.todo.name.toLowerCase().includes('bitcoin')
      ) {
        return alert("Nope. That's a bad idea.")
      }

      if (
        action.type === ADD_GOAL &&
        action.goal.name.toLowerCase().includes('bitcoin')
      ) {
        return alert("Nope. That's a bad idea.")
      }

      return next(action)
    }
```

Note that `const checker = (store) => (next) => (action) => {}` is equivalent to 

```js
function checker(store) {
  return function (next) {
    return function (action){
      ...
    }
  }
}
```

2. change `createStore` so that it knows we are applying middleware

```js
const store = Redux.createStore(Redux.combineReducers({
      todos,
      goals,
    }), Redux.applyMiddleware(checker))
```

```js
applyMiddleware(...middlewares)
```

Note the spread operator on the `middlewares` parameter. This means that we can pass in as many different middleware as we want! Middleware is called in the order in which they were provided to `applyMiddleware()`.

3. For all dispatch call, change it back to original like:

```js
store.dispatch(addTodoAction({
        name,
        complete: false,
        id: generateId()
}))
```

https://github.com/udacity/reactnd-redux-todos-goals/blob/e07e1785e0bf90b2128eac3a63674ec9c8daabac/index.html

Next, add new middleware to log information:

```js
 const logger = (store) => (next) => (action) => {
       console.group(action.type)
         console.log('The action: ', action)
         const result = next(action)
         console.log('The new state: ', store.getState())
       console.groupEnd()
       return result
     }
```

```js
const store = Redux.createStore(Redux.combineReducers({
       todos,
       goals,
     }), Redux.applyMiddleware(checker, logger))
```

#### Add React into UI

1. first indclude two more scripts. `babel`is to convert css to javascript.

```js
<!DOCTYPE html>
<html>
<head>
  <title>Udacity Todos Goals</title>
  <script src='https://cdnjs.cloudflare.com/ajax/libs/redux/3.7.2/redux.min.js'></script>
  <script src='https://unpkg.com/react@16.3.0-alpha.1/umd/react.development.js'></script>
  <script src='https://unpkg.com/react-dom@16.3.0-alpha.1/umd/react-dom.development.js'></script>
  <script src='https://unpkg.com/babel-standalone@6.15.0/babel.min.js'></script>
</head>
<body>
  <div>
    <h1>Todo List</h1>
    <input id='todo' type='text' placeholder='Add Todo' />
    <button id='todoBtn'>Add Todo</button>
    <ul id='todos'></ul>
  </div>
  <div>
    <h1>Goals</h1>
    <input id='goal' type='text' placeholder='Add Goal' />
    <button id='goalBtn'>Add Goal</button>
    <ul id='goals'></ul>
  </div>

  <hr /> **** add a separation line *****

  <div id='app'></div> ********* to tell react where to mount to *********
  
   <script type='text/javascript'>
     .... old redux code
   </script>
  
   <script type='text/babel'>  *********** add react ******************
    function List (props) { *** take array and create a list
      return (
        <ul>
          <li>LIST</li>
        </ul>
      )
    }

    class Todos extends React.Component {
      render() {
        return (
          <div>
            TODOS

            <List />
          </div>
        )
      }
    }

    class Goals extends React.Component {
      render() {
        return (
          <div>
            GOALS

            <List />
          </div>
        )
      }
    }

    class App extends React.Component {
      render() {
        return (
          <div>
            <Todos />
            <Goals />
          </div>
        )
      }
    }

    ReactDOM.render(
      <App />,
      document.getElementById('app')
    )
  </script>
</body>
</html>
```

**Next and most importantly is how we can add Redux into React:**

- where the `store.dispatch()` code goes in a React component
- how a React component is passed the Redux store as a prop



```js
<!DOCTYPE html>
<html>
<head>
  <title>Udacity Todos Goals</title>
  <script src='https://cdnjs.cloudflare.com/ajax/libs/redux/3.7.2/redux.min.js'></script>
  <script src='https://unpkg.com/react@16.3.0-alpha.1/umd/react.development.js'></script>
  <script src='https://unpkg.com/react-dom@16.3.0-alpha.1/umd/react-dom.development.js'></script>
  <script src='https://unpkg.com/babel-standalone@6.15.0/babel.min.js'></script>
</head>
<body>
  <div id='app'></div>

  <script type='text/javascript'>
    function generateId () {
      return Math.random().toString(36).substring(2) + (new Date()).getTime().toString(36);
    }

    // App Code
    const ADD_TODO = 'ADD_TODO'
    const REMOVE_TODO = 'REMOVE_TODO'
    const TOGGLE_TODO = 'TOGGLE_TODO'
    const ADD_GOAL = 'ADD_GOAL'
    const REMOVE_GOAL = 'REMOVE_GOAL'

    function addTodoAction (todo) {
      return {
        type: ADD_TODO,
        todo,
      }
    }

    function removeTodoAction (id) {
      return {
        type: REMOVE_TODO,
        id,
      }
    }

    function toggleTodoAction (id) {
      return {
        type: TOGGLE_TODO,
        id,
      }
    }

    function addGoalAction (goal) {
      return {
        type: ADD_GOAL,
        goal,
      }
    }

    function removeGoalAction (id) {
      return {
        type: REMOVE_GOAL,
        id,
      }
    }

    function todos (state = [], action) {
      switch(action.type) {
        case ADD_TODO :
          return state.concat([action.todo])
        case REMOVE_TODO :
          return state.filter((todo) => todo.id !== action.id)
        case TOGGLE_TODO :
          return state.map((todo) => todo.id !== action.id ? todo :
            Object.assign({}, todo, { complete: !todo.complete }))
        default :
          return state
      }
    }

    function goals (state = [], action) {
      switch(action.type) {
        case ADD_GOAL :
          return state.concat([action.goal])
        case REMOVE_GOAL :
          return state.filter((goal) => goal.id !== action.id)
        default :
          return state
      }
    }

    const checker = (store) => (next) => (action) => {
      if (
        action.type === ADD_TODO &&
        action.todo.name.toLowerCase().includes('bitcoin')
      ) {
        return alert("Nope. That's a bad idea.")
      }

      if (
        action.type === ADD_GOAL &&
        action.goal.name.toLowerCase().includes('bitcoin')
      ) {
        return alert("Nope. That's a bad idea.")
      }

      return next(action)
    }

    const logger = (store) => (next) => (action) => {
      console.group(action.type)
        console.log('The action: ', action)
        const result = next(action)
        console.log('The new state: ', store.getState())
      console.groupEnd()
      return result
    }

    const store = Redux.createStore(Redux.combineReducers({
      todos,
      goals,
    }), Redux.applyMiddleware(checker, logger))
  </script>

  <script type='text/babel'>
    function List (props) {
      return (
        <ul>
          {props.items.map((item) => (
            <li key={item.id}>
              <span
                onClick={() => props.toggle && props.toggle(item.id)}
                style={{textDecoration: item.complete ? 'line-through' : 'none'}}>
                  {item.name}
              </span>
              <button onClick={() => props.remove(item)}>
                X
              </button>
            </li>
          ))}
        </ul>
      )
    }

    class Todos extends React.Component {
      addItem = (e) => {
        e.preventDefault()
        const name = this.input.value
        this.input.value = ''

        this.props.store.dispatch(addTodoAction({
          name,
          complete: false,
          id: generateId()
        }))
      }
      removeItem = (todo) => {
        this.props.store.dispatch(removeTodoAction(todo.id))
      }
      toggleItem = (id) => {
        this.props.store.dispatch(toggleTodoAction(id))
      }
      render() {
        return (
          <div>
            <h1>Todo List</h1>
            <input
              type='text'
              placeholder='Add Todo'
              ref={(input) => this.input = input}
            />
            <button onClick={this.addItem}>Add Todo</button>

            <List
              toggle={this.toggleItem}
              items={this.props.todos}
              remove={this.removeItem}
            />
          </div>
        )
      }
    }

    class Goals extends React.Component {
      addItem = (e) => {
        e.preventDefault()
        const name = this.input.value
        this.input.value = ''

        this.props.store.dispatch(addGoalAction({
          id: generateId(),
          name,
        }))
      }
      removeItem = (goal) => {
        this.props.store.dispatch(removeGoalAction(goal.id))
      }
      render() {
        return (
          <div>
            <h1>Goals</h1>
            <input
              type='text'
              placeholder='Add Goal'
              ref={(input) => this.input = input} //ref to DOM element, update state
            />
            <button onClick={this.addItem}>Add Goal</button>

            <List //for diaplay
              items={this.props.goals}
              remove={this.removeItem}
            />
          </div>
        )
      }
    }

    class App extends React.Component {
      componentDidMount () {
        const { store } = this.props

        store.subscribe(() => this.forceUpdate()) // do not need to set state
      }
      render() {
        // below two lines will run whenever component is rerendered. 
        const { store } = this.props
        const { todos, goals } = store.getState()

        return (
          <div>
            <Todos todos={todos} store={this.props.store} />//important to pass store
            <Goals goals={goals} store={this.props.store} />
          </div>
        )
      }
    }

    ReactDOM.render(
      <App store={store}/>, // pass initial store here
      document.getElementById('app')
    )
  </script>
</body>
</html>
```

----

##### Topic: what is promise?

> The promise object is used for deferred and asynchronous computations

Call back function is a function that passed to a function and will be invoked at a later time.

```js
function loadImage(src, parent, callback) {
	var img = document.createElement('img');
	img.src = src;
	img.onload = callback;
	parent.appendChild(img);
}
```

- But problem is how do you handle errors? if error at second line, do you still assign callback?

- Another problem is called "Pyramid of Dooms". Its that there is another callback needed after one callback and another after one … Are we really going to pass callback function into callback and so on?

Three states of promise:

	1. Fulfilled 
 	2. Rejected
 	3. Pending
 	4. Settled: either fulfilled or rejected

- Compared `promise` with `listener`. If event already fires, listener will not invoke anymore. but promise might. 

- `Promise` can only be resolved once to `fulfilled`. While events can fire multiple times.

- `Promise` might still block the work if the time from promise created to resolved is long...

![image-20200325010327123](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325010327123.png)

Solution:

1. ajax is definitely good use case
2. should not use promise for large work in main thread. promise is in the main thread, so you do not get any benefit
3. should not use to create a lot of htmls elements. This is asychronous already. no need.
4. Posting message in the web worker. web worker is used when you have a huge work and you can create a new thread solely to do that.

---



#### Redux for Asychronous API Call

1. include new script, to mimic we have a server

   `<script src="https://tylermcginnis.com/goals-todos-api/index.js"></script>`

   You can type `API` in console to see more information. 

   Let's examine a part of code here:

   ```js
   API.fetchTodos = function () {
     return new Promise((res, rej) => {
       setTimeout(function () {
         res(todos);
       }, 2000);
     });
   };
   ```

   

2. Check those commits: https://github.com/udacity/reactnd-redux-todos-goals/commit/98d9b5468262eb4ea786cb55c3d68ed9de78af09 

   When component is rendered, api calls are called to fetch all todo and goals. This is asyc api call so that user does not need to pause and wait for fetching. 

   ```js
   Promise.all([
     API.fetchTodos(),
     API.fetchGoals()
   ]).then(([ todos, goals ]) => {
     store.dispatch(receiveDataAction(todos, goals))
   })
   ```

   A new action `receveDataAction` is dispatched and handled in the reducer functions to return todo and goals. Now, when you enter url or refresh the page, after a few seconds, all todos and goals are listed below.

3. But since the data appears after a few seconds, we would like to add loading text.

   Soln: make a new reducer to handle loadings … 

   ```js
   function loading (state = true, action) {
     switch(action.type) {
       case RECEIVE_DATA:
         return false
       default
         return state
     }
   }
   ```

   Then add `loading` into `Redux.createStore(Redux.combineReducers( ...))`

   See the commit: https://github.com/udacity/reactnd-redux-todos-goals/commit/6bb7f0d521986a316b9aa45cac4f2fae886c4bfc

3. When user clicks remove, we actually need to remove data:

   ```js
   removeItem = (todo) => {
     return API.deleteTodo(todo.id)
      .then(() => {
       this.props.store.dispatch(removeTodoAction(todo.id))
     })
   }
   ```

   Right now the order is: first api call to remove, then clear item on UI. This could result a small delay. We can reorder these to clear ui first and then send request. we will use `.catch` to catch all the errors such that if any error occurs, we add whatever removed back.

   ```js
   removeItem = (todo) => {
     this.props.store.dispatch(removeTodoAction(todo.id))
   
     return API.deleteTodo(todo.id)
       .catch(() => {
       this.props.store.dispatch(addTodoAction(todo))
       alert('An error occurred. Try again.')
     })
   }
   ```

   Same thing for toggle method:

   ```js
   toggleItem = (id) => {
     this.props.store.dispatch(toggleTodoAction(id))
   
     return API.saveTodoToggle(id)
       .catch(() => {
       this.props.store.dispatch(toggleTodoAction(id))
       alert('An error occurred. Try again.')
     })
   }
   ```

   But save method, we cannot do optimistic updates because we need backend to generate id for new items now and return it to us.

   ```js
   class Todos extends React.Component {
     addItem = (e) => {
       e.preventDefault()
   
       return API.saveTodo(this.input.value)
         .then((todo) => {
         this.props.store.dispatch(addTodoAction(todo))
         this.input.value = ''
       })
         .catch(() => {
         alert('There was an error. Try again.')
       })
     }
   ```

   > ### Optimistic Updates
   >
   > When dealing with asynchronous requests, there will always be some  delay involved. If not taken into consideration, this could cause some  weird UI issues. For example, let’s say when a user wants to delete a  todo item, that whole process from when the user clicks“delete” to when  that item is removed from the database takes two seconds. If you  designed the UI to *wait for the confirmation from the server* to remove the item from the list on the client, your user would click  “delete” and then would have to wait for two seconds to see that update  in the UI. That’s not the best experience.
   >
   > Instead what you can do is a technique called **optimistic updates**. Instead of waiting for confirmation from the server, just instantly  remove the user from the UI when the user clicks “delete”, then, if the  server responds back with an error that the user wasn’t actually  deleted, you can add the information back in. This way your user gets  that instant feedback from the UI, but, under the hood, the request is  still asynchronous. 

### THUNK

Our current api is mixing api calls with ui logics. we would like to keep them seperated. And put api call into action creator.

![image-20200407150809701](/Users/gabriel/Library/Application Support/typora-user-images/image-20200407150809701.png)

For example, we want to change our following code:

```js
removeItem(item) {
  const { dispatch } = this.props.store

  dispatch(removeTodoAction(item.id))

  return API.deleteTodo(item.id)
    .catch(() => {
      dispatch(addTodoAction(item))
      alert('An error occured. Try again.')
    })
  }
}
```

to ...

```js
removeItem(item) {
  const { dispatch } = this.props.store

  return dispatch(handleDeleteTodo(item)) //handleDeleteTodo is an action creator 
}
```

The `removeItem()` function only has one task; dispatching that a specific item needs to be deleted. 

<u>However, we need to make it so our `handleDeleteTodo` action creator makes an asynchronous request before it returns the action. What if we just return a promise from `handleDeleteTodo` that resolves with the action once we get the data? Well, that won't  quite work; as of right now, every action creator needs to return an *object*, not a promise:</u>

```js
function handleDeleteTodo (todo) {
  // need to dispath & call api and dispatch again if there's error
  // soln: return a function takes a dispatch
  return (dispatch) => {
    // below is just copied over
    dispatch(removeTodoAction(todo.id))

    return API.deleteTodo(todo.id)
      .catch(() => {
      dispatch(addTodoAction(todo))
      alert('An error occurred. Try again.')
    })
  }
}
```

The reducer expects to receive an action object, but what if, instead of returning an object, we have our action creator return a function?

We could use some ==middleware== to check if the returned action is  either a function or an object. If the action is an object, then things  will work as normal - it will call the reducer passing it the action.  However, if the action is a function, it can invoke the function and  pass it whatever information it needs (e.g. a reference to the `dispatch()` method). This function could do anything it needs to do, like making asynchronous network requests, and can then dispatch a *different* action (that returns a regular object) when its finished.

```js
const thunk = (store) => (next) => (action) => {
  if (typeof action === 'function') {
    return action(store.dispatch)
  } else {
    return next(action)
  }
}
```

Alternatively, since thunk is so common, we could just use a public library ...

```js
<script src="https://unpkg.com/redux-thunk@2.2.0/dist/redux-thunk.min.js"></script>
```

```js
const store = Redux.createStore(Redux.combineReducers({
      todos,
      goals,
      loading,
}), Redux.applyMiddleware(ReduxThunk.default, checker, logger))
```

==Without thunks, synchronous dispatches are the default. We *could* still make API calls from React components (e.g., using the `componentDidMount()` lifecycle method to make these requests) -- but using thunk middleware  gives us a cleaner separation of concerns. Components don't need to  handle what happens after an asynchronous call, since API logic is *moved away* from components to action creators. This also lends itself to greater  predictability, since action creators will become the source of every  change in state. With thunks, we can dispatch an action only when the  server request is resolved!==



Lastly, remember the below chunk of code to initially fetch all todos and goals, we would like to move them into an action creator as well.

```js
Promise.all([
  API.fetchTodos(),
  API.fetchGoals()
]).then(([ todos, goals ]) => {
  store.dispatch(receiveDataAction(todos, goals))
})
```

So instead of above, we write `store.dispatch(handleInitialData())`

```js
funciton handleInitialData() {
  return (dispatch) {
    Promise.all([
 			API.fetchTodos(),
  		API.fetchGoals()
		]).then(([ todos, goals ]) => {
  		dispatch(receiveDataAction(todos, goals))
		})
  }
}
```

**[Complete Code]** https://github.com/udacity/reactnd-redux-todos-goals/blob/e40512f5cd22b35c6461fa334636aaa1eb9f27d2/index.html

- [Redux Promise](https://github.com/redux-utilities/redux-promise) - FSA-compliant promise middleware for Redux.
- [Redux Saga](https://github.com/redux-saga/redux-saga) - An alternative side effect model for Redux apps

---

### React & Redux

Right now, redux we learnt is decoupled with react. we want them to work more closely.

**Problem 1:** its tough for component to access store. We have to pass the store down as a prop but this is inefficient in large app with dozens of components.

```js
ReactDOM.render(
  <App store={store}/>,
  document.getElementById('app')
)
```

**Problem 2:** Redux is popular for its efficiency in knowing when sth has changed and rerender that part of UI. So we need to find a way to re-render only if hte data depend on (from the store) changes. We currently solve this by calling `getstate` at root and then pass the data down. This does not scale.

```js
class App extends React.Component {
  componentDidMount () {
    const { store } = this.props
    store.dispatch(handleInitialData())
    store.subscribe(() => this.forceUpdate())
  }
  render() {
    const { store } = this.props
    const { todos, goals, loading } = store.getState()

    if (loading === true) return <h3>Loading</h3>
    return (
      <div>
        <Todos todos={todos} store={this.props.store} />
        <Goals goals={goals} store={this.props.store} />
       </div>
		)
	}
}
```

Assume that we have a component tree such that `parent component` renders `child component` which renders `grandchild component` which display a variable passed from `parent` . Then even though child does not need that variable, it stills needs to carry it in order to pass that to `grandchild`. We can avoid this using ==Context API==

Add this line first: `const Context = React.createContext();`

Context api has two parts: `Context.Provider` & `Context.Consumer`

Use `provider`in the upper level of component tree where data is provided to be passed.

```js
class App extends React.Component {
  render() {
  const name = 'Tyler';

  return (
    <Context.Provider value={name}> // pass data name
      <Parent />
    </Context.Provider>
    );
  }
}
```

Then we do nothing in `child component`, neither needs to pass the variable. But at the component that we are ready to use `name`, we will apply `Consumer`:

```js
function Grandchild ({ name }) {
  return (
    <Context.Consumer>
      {(name) => (
        <div>
          <h1>Grandchild</h1>
          <h3>Name: {name}</h3>
        </div>
      )}
    </Context.Consumer>
  );
}
```

**We should pass consumer a function as a child that accepts the variable and return JSX.**

```js
class Goals extends React.Component {
  addItem = (e) => {
    e.preventDefault()
    this.props.dispatch(handleAddGoal(
      this.input.value,
      () => this.input.value = ''
    ))
  }
  removeItem = (goal) => {
    this.props.dispatch(handleDeleteGoal(goal))
  }
  render() {
    return (
      <div>
      <h1>Goals</h1>
      <input
      type='text'
      placeholder='Add Goal'
      ref={(input) => this.input = input}
/>
<button onClick={this.addItem}>Add Goal</button>

<List
items={this.props.goals}
remove={this.removeItem}
/>
  </div>
)
}
}

class ConnectedGoals extends React.Component {
  render() {
    return (
      <Context.Consumer>
      {(store) => {
      const { goals } = store.getState()

      return <Goals goals={goals} dispatch={store.dispatch} />
    }}
      </Context.Consumer>
    )
  }
}

class App extends React.Component {
  componentDidMount () {
    const { store } = this.props

    store.dispatch(handleInitialData())

    store.subscribe(() => this.forceUpdate())
  }
  render() {
    const { store } = this.props
    const { loading } = store.getState()

    if (loading === true) {
      return <h3>Loading</h3>
    }

    return (
      <div>
        <ConnectedTodos /> //!!!
        <ConnectedGoals /> //!!!
      </div>
    )
  }
}

class ConnectedApp extends React.Component {
  render() {
    return (
      <Context.Consumer>
      {(store) => (
      <App store={store} /> //!!!
		)}
  	</Context.Consumer>
		)
	}
}

const Context = React.createContext()

class Provider extends React.Component { //!!!
  render () {
    return (
      <Context.Provider value={this.props.store}>
      	{this.props.children} 
				// check https://reactjs.org/docs/composition-vs-inheritance.html
				//everything inside <Provider> </Provider> just passed as props.children
			</Context.Provider>
		)
	}
}

ReactDOM.render(
  <Provider store={store}>
  	<ConnectedApp /> // before it is just <App>
  </Provider>,
	document.getElementById('app')
)
```

Above is the `Presentational and Container Components` The reason we create ConnectedApp is to separate data from rendering UI. Connected app is used solely to get data and Goals/Todos are sole for rendering UI.

Its weird to have so many connected components. So we will create a helper function connect such that we can replace components with usage of this function.

Convert 

```js
 class ConnectedTodos extends React.Component {
   render() {
     return (
       <Context.Consumer>
       {(store) => {
       const { todos } = store.getState()

       return <Todos todos={todos} dispatch={store.dispatch} />
     }}
       </Context.Consumer>
     )
   }
 }
```

to ...

```js
const ConnectedTodos = connect((state) => ({
  todos: state.todos
}))(Todos)
```

And convert 

```js
class ConnectedApp extends React.Component {
	render() {
		return (
      <Context.Consumer>
      	{(store) => (
      		<App store={store} />
				)}
			</Context.Consumer>
		)
	}
}
```

to ...

```js
const ConnectedApp = connect((state) => ({
  loading: state.loading
}))(App)

// below is just to read, no changes
const Context = React.createContext()

class Provider extends React.Component {
  render () {
    return (
      <Context.Provider value={this.props.store}>
      	{this.props.children}
			</Context.Provider>
		)
	}
}

ReactDOM.render(
  <Provider store={store}>
  	<ConnectedApp />
  </Provider>,
	document.getElementById('app')
)
```

**It is important to know that ConnectedApp is a function that takes a regular(presentational) react component and return a new connected component**

and then create library function `connect`

```js
function connect(mapStateToProps) { //arg is a function
  return (Component) => { //component is passed App or whatever

    class Receiver extends React.Component {

      componentDidMount() {
        const {subscribe} = this.props.store;
        this.unsubscribe = subscribe(() =>{
          this.forceUpdate();
        })
      }

      componentWillUnmount(){
        this.unsubscribe();
      }

      render() {
        const {dispatch, getState } = this.props.store;
        const state = getState();
        const stateNeeded = mapStateToProps(state); //get {loading : ...}
        return <Component {...stateNeeded} dispatch={dispatch}/>
      }
    }

		//responsible for getting data
    class ConnectedComponent extends React.Component {
        render() {
          return (
            <Context.Consumer>
            	{store => <Receiver store={store}/>} //receiver is for rendering ui
            </Context.Consumer>
          )
        }

      }

    return ConnectedComponent; //return a component
  }
}
```

Ofc, there will be changes in the App component

```js
    class App extends React.Component {

        componentDidMount = () => {
            const {store} = this.props;
            //store.subscribe(() => this.forceUpdate()); NOT NEED            		
            const { dispatch } = this.props;
            dispatch(handleInitialData());
        };

        render() {
            const {store} = this.props;
            const {loading} = store.getState();

            // if (loading) { REPLACED WITH BELOW
            if (this.props.loading) {
                return (
                    <h3>Loading . . .</h3>
                )
            }
            return (
                <div>
                    <ConnectedTodos/>
                    <ConnectedGoals/>
                </div>
            )
        }
    }
```

[code]: https://github.com/ahdeshpande/Redux-Intro/commit/c5244908e6eb74bdd003635734437297cc92a9e7

In fact, `connect` function is commonly used and has official library `react-redux` for us to use.

```js
<script src="https://unpkg.com/react-redux@5.0.6/dist/react-redux.min.js"></script>
```

It provides `provider` `connect `

change ` const ConnectedApp = connect((state) => ({` 

to `    const ConnectedApp = ReactRedux.connect((state) => ({`

Also change`<Provider store={store}>` to `<ReactRedux.Provider store={store}>`

[code]: https://github.com/udacity/reactnd-redux-todos-goals/commit/e6ab31c60bcac704f05b21627594328e56478efd

---

### Folder Structure

```
├── dashboard
│ ├── actions.js
│ ├── index.js
│ └── reducer.js
└── nav
 ├── actions.js
 ├── index.js
 └── reducer.js
```

Code is all in one file. we will sue `create-react-app` to scaffold out a React app for us.

`$ npm install -g create-react-app`

`$ create-react-app udacity-goals-todos`

`$ cd XXXX`

`$ yarn add goals-todos-api redux react-redux redux-thunk`

`$ yarn start`

- next, delete some files, `logo` `serviceworker` `app.test.js` `app.css`

- make a `component`  and `action` folder in root

- save `todo.js` `goal.js` and `share.js` in `/action` to store those actions ==[do not forget to export functions]==

- create a folder `reducer`and store `goal.js` `todo.js` `loading.js` and `index.js` which is for us to use the `combineReducer` method so we dont need to do this in the main index.js file

  ```js
  //index.js
  import { combineReducers } from 'redux'
  
  import todos from './todos'
  import loading from './loading'
  import goals from './goals'
  
  export default combineReducers({
    todos,
    loading,
    goals,
  }) // do not worry about middleware, we do it in middleware index.js file
  ```

- Create folder `middleware` to store `checker.js `  `logge.js` and `index.js`

  don't forget to `expoert default logger`

  ```js
  //index.js
  import checker from './checker'
  import logger from './logger'
  import thunk from 'redux-thunk'
  import { applyMiddleware } from 'redux'
  
  export default applyMiddleware(
    thunk,
    checker,
    logger
  ) 
  ```

- In `/component` creates `app.js` `list.js` `goals.js` `todos.js` `index.js`

- 

