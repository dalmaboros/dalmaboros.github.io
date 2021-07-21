---
layout:     post
title:      "Choosing Redux State vs React State"
date:       2021-07-21 09:11:00 -0400
permalink:  redux_state_vs_react_state
---

I recently built a React-Redux app for my final Flatiron School project. The project requirements specify that Redux must be used to manage state. What the requirements don't specify is that Redux doesn't need to be used manage ALL state.

As it turns out, there was a part of my app that was managed by Redux that didn't need to be, and so I changed that. 

How do I know what state needs to be managed by Redux and what doesn't?

## Props

I'll start answering this question with a recap of what state is. I find it helps to understand state by also understanding props. Both state and props are objects that hold data. Props--short for properties--are passed from a parent React component to a child React component. A child component receiving props cannot change these props. These props can only be changed by the parent component. Alternatively, the child component can store the value of a received prop in its state, which it can then modify. 

## React State

What is state, though? Like props, state is an object that holds data. Unlike props, this data is not passed from a parent component to a child component. In React, state is private to a component, and it is created, updated, and used within that component. So unlike props, state can be changed by the component in which it is contained, and in fact, change is kind of the whole point of state. A component may be stateless, and by default it is. If there is no information that needs to be stored or modified by the component, then we simply don't give it state. In a stateful React component, state is accessible only to the component itself (although values from a component's state can be passed as props to child components).

## Redux State

So in React, state is local to components. But middleware exists that enables state to be kept in a central store, accessible to all components in an application. Redux is one such middleware. It enables any component to connect to the central store that holds the entire application's state. This is certainly useful when multiple components need access to the same data stored in state, but when a component has state that no other part of the application needs to access, Redux is not necessary. 

In the words of the Dan Abramov, co-creator of Redux, ["Local state is fine."](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)

## Moving State From Redux to React

There was one such component in my application that used Redux middleware to store and access some state to which no other component needed access. The `BrowsersContainer` component contains a modal that is conditionally displayed based on the states of two of its child components. Each of the two child `ClothesBrowser` components displays an image of a piece of clothing--one of a top and one of a bottom. If the pieces of clothing in the two `ClothesBrowser`s do not match each other, a modal with the text "Mis-Match" is displayed on screen when the user clicks a button labeled "Dress Me." The modal's state of visibility (visible or hidden) was being stored in the Redux store, making it accessible to all components connected to the store. Not a single other component, however, was accessing that modal's state of visibility. So I could go ahead and move that data's management from the global Redux store to local state in the `BrowsersContainer` component.

## Code Refactoring

This is what the `BrowsersContainer` originally looked like:

```js
// BrowsersContainer.js
import React, { Component } from 'react'
import { connect } from 'react-redux'
import { Link } from 'react-router-dom'
import ClothesBrowser from '../components/ClothesBrowser'
import Modal from '../components/Modal'
import { hideModal, showModal } from '../actions/clothesActions'
import './BrowsersContainer.css'

class BrowsersContainer extends Component {
    handleOnClick = () => {
        if (this.isMatch()) {
            this.props.hideModal()
        } else {
            this.props.showModal()
        }
    }

    isMatch = () => {
        let selectedTop = this.props.clothes.tops.pieces[this.props.clothes.tops.selectedPiece]
        let selectedBottom = this.props.clothes.bottoms.pieces[this.props.clothes.bottoms.selectedPiece]

        if (selectedTop.image_url === "https://i.imgur.com/LH4eU3x.jpg" && selectedBottom.image_url === "https://i.imgur.com/5RGZE6c.jpg") {
            return true
        } else {
            return false
        }
    }

    handleClose = () => {
        this.props.hideModal()
    }

    render() {
        if (this.props.clothes.length === 0) return null;

        let dressMePath;
        if (this.isMatch()) {
            dressMePath = "/dressme"
        } else {
            dressMePath = "#"
        }

        return (
            <div className="main-container">

                <Modal show={this.props.modalVisible} handleClose={this.handleClose} />

                <div className="interface-column col-1">
                    <div className="button">
                        <Link to="/browse">
                            <button className="button-nav active">Browse</button>
                        </Link>
                    </div>
                </div>

                <div className="interface-column col-2">
                    <div className="content-wrapper">
                        <ClothesBrowser 
                            pieces={this.props.clothes.tops.pieces} 
                            category={this.props.clothes.tops.category} 
                            selectedPiece={this.props.clothes.tops.selectedPiece} />
                        <ClothesBrowser 
                            pieces={this.props.clothes.bottoms.pieces} 
                            category={this.props.clothes.bottoms.category} 
                            selectedPiece={this.props.clothes.bottoms.selectedPiece} />
                    </div>
                </div>

                <div className="interface-column col-3">
                    <div className="button">
                        <Link to={dressMePath}>
                            <button className="button-nav" onClick={this.handleOnClick}>Dress <br></br>Me</button>
                        </Link>
                    </div>
                </div>

            </div>
        )
    }
}

const mapStateToProps = state => {
    return state
}
  
const mapDispatchToProps = dispatch => {
    return {
        hideModal: () => dispatch(hideModal()),
        showModal: () => dispatch(showModal())
    }
}
  
export default connect(mapStateToProps, mapDispatchToProps)(BrowsersContainer);

```

These are the steps I took to move the modal's visibility state management from the Redux store to the `BrowsersContainer`: 

Since I want to add local state to this component, I add it to the `constructor()` method, which is called automatically when a class object is initialized. In this constructor method, `super()` calls the constructor of the parent class (superclass), React's `Component` class. Then some key value pairs are defined in the initial state object.

```js
// BrowsersContainer.js
...
class BrowsersContainer extends Component {
    constructor(props) {
        super(props)
        this.state = {
            isMatch: false,
            modalVisible: false,
            dressMePath: "#"
        }
    }
...
```


I still want to handle the click event with `handleOnClick`, but now I'll use one piece of the local state, `isMatch`, to set another piece of the local state, `modalVisible`. 


```js
// BrowsersContainer.js
...
handleOnClick = () => {
    if (this.state.isMatch) {
        this.setState({modalVisible: false})
    } else {
        this.setState({modalVisible: true})
    }
}
...
```


Where then, is `isMatch` determined now? I move the logic of the old `isMatch()` function into React's `static getDerivedStateFromProps()` lifecycle method, and this is where the `isMatch` part of state is set:


```js
// BrowsersContainer.js
...
static getDerivedStateFromProps(props, state) {
    if (props.clothes.tops && props.clothes.bottoms) {
        let selectedTop = props.clothes.tops.pieces[props.clothes.tops.selectedPiece]
        let selectedBottom = props.clothes.bottoms.pieces[props.clothes.bottoms.selectedPiece]

        if (selectedTop.image_url === "https://i.imgur.com/LH4eU3x.jpg" && selectedBottom.image_url === "https://i.imgur.com/5RGZE6c.jpg") {
            return {isMatch: true, dressMePath: "/dressme"}
        } else {
            return {isMatch: false, dressMePath: "#"}
        }
    } else {
        return null
    }
}
...
```


In addition to `isMatch`, `dressMePath` is also being set here. The value of `dressMePath` is used as the href attribute value of the 'Dress Me' button's anchor. Before every call to `render()`, the 'Dress Me' button's link path is dynamically set depending on whether the outfit is a match ("#" if not a match, so the href leads nowhere, and "/dressme" if it is a match).

The old `isMatch()` function is deleted and the call to `isMatch()` that was in `render()` is removed.

The `handleClose` method is modified to instead of calling a dispatch, to set the `modalVisible` part of state to false:


```js
// BrowsersContainer.js
...
handleClose = () => {
    this.setState({modalVisible: false})
}
...
```

5. `mapDispatchToProps` can now be removed entirely, since nothing is dispatched from this component anymore. We can also stop importing the `hideModal` and `showModal` action creators.

The final refactored `BrowsersContainer` component looks like this:

```js
// BrowsersContainer.js
import React, { Component } from 'react'
import { connect } from 'react-redux'
import { Link } from 'react-router-dom'
import ClothesBrowser from '../components/ClothesBrowser'
import Modal from '../components/Modal'
import './BrowsersContainer.css'

class BrowsersContainer extends Component {
    constructor(props) {
        super(props)
        this.state = {
            isMatch: false,
            modalVisible: false,
            dressMePath: "#"
        }
    }

    handleOnClick = () => {
        if (this.state.isMatch) {
            this.setState({modalVisible: false})
        } else {
            this.setState({modalVisible: true})
        }
    }

    handleClose = () => {
        this.setState({modalVisible: false})
    }

    static getDerivedStateFromProps(props, state) {
        if (props.clothes.tops && props.clothes.bottoms) {
            let selectedTop = props.clothes.tops.pieces[props.clothes.tops.selectedPiece]
            let selectedBottom = props.clothes.bottoms.pieces[props.clothes.bottoms.selectedPiece]

            if (selectedTop.image_url === "https://i.imgur.com/LH4eU3x.jpg" && selectedBottom.image_url === "https://i.imgur.com/5RGZE6c.jpg") {
                return {isMatch: true, dressMePath: "/dressme"}
            } else {
                return {isMatch: false, dressMePath: "#"}
            }
        } else {
            return null
        }
    }

    render() {
        if (this.props.clothes.length === 0) return null;

        return (
            <div className="main-container">

                <Modal show={this.state.modalVisible} handleClose={this.handleClose} />

                <div className="interface-column col-1">
                    <div className="button">
                        <Link to="/browse">
                            <button className="button-nav active">Browse</button>
                        </Link>
                    </div>
                </div>

                <div className="interface-column col-2">
                    <div className="content-wrapper">
                        <ClothesBrowser 
                            pieces={this.props.clothes.tops.pieces} 
                            category={this.props.clothes.tops.category} 
                            selectedPiece={this.props.clothes.tops.selectedPiece} />
                        <ClothesBrowser 
                            pieces={this.props.clothes.bottoms.pieces} 
                            category={this.props.clothes.bottoms.category} 
                            selectedPiece={this.props.clothes.bottoms.selectedPiece} />
                    </div>
                </div>

                <div className="interface-column col-3">
                    <div className="button">
                        <Link to={this.state.dressMePath}>
                            <button className="button-nav" onClick={this.handleOnClick}>Dress <br></br>Me</button>
                        </Link>
                    </div>
                </div>

            </div>
        )
    }
}

const mapStateToProps = state => {
    return state
}
  
export default connect(mapStateToProps)(BrowsersContainer);
```


Since the `hideModal` and `showModal` action creators are no longer used, they can be deleted from `clothesActions.js`, so this:


```js
// clothesActions.js

export const fetchClothes = () => {
    return (dispatch) => {
        dispatch({type: 'LOADING_CLOTHES'})
        let clothes = {}
        fetch('/api/v1/tops')
            .then(response => response.json())
            .then(tops => {
                clothes = {tops}
                return fetch('/api/v1/bottoms')
            })
            .then(response => response.json())
            .then(bottoms => {
                clothes = {
                    ...clothes,
                    bottoms
                }
                dispatch({type: 'LOAD_CLOTHES', payload: clothes})
            })
    }
}

export const selectNextPiece = (category) => {
    return(dispatch) => {
        dispatch({type: 'SELECT_NEXT_PIECE', category: category})
    }
}

export const selectPreviousPiece = (category) => {
    return(dispatch) => {
        dispatch({type: 'SELECT_PREVIOUS_PIECE', category: category})
    }
}

// Delete these two action creators
export const showModal = () => {
    return(dispatch) => {
        dispatch({type: 'SHOW_MODAL'})
    }
}

export const hideModal = () => {
    return(dispatch) => {
        dispatch({type: 'HIDE_MODAL'})
    }
}
```


becomes this:


```js
// clothesActions.js

export const fetchClothes = () => {
    return (dispatch) => {
        dispatch({type: 'LOADING_CLOTHES'})
        let clothes = {}
        fetch('/api/v1/tops')
            .then(response => response.json())
            .then(tops => {
                clothes = {tops}
                return fetch('/api/v1/bottoms')
            })
            .then(response => response.json())
            .then(bottoms => {
                clothes = {
                    ...clothes,
                    bottoms
                }
                dispatch({type: 'LOAD_CLOTHES', payload: clothes})
            })
    }
}

export const selectNextPiece = (category) => {
    return(dispatch) => {
        dispatch({type: 'SELECT_NEXT_PIECE', category: category})
    }
}

export const selectPreviousPiece = (category) => {
    return(dispatch) => {
        dispatch({type: 'SELECT_PREVIOUS_PIECE', category: category})
    }
}
```


We can thus delete the `'SHOW_MODAL'` AND `'HIDE_MODAL'` cases from `clothesReducer.js`. We can also delete `modalVisible` from `initialState`:


```js
// clothesReducer.js

const initialState = { 
    clothes: [],
    loading: false,
    // This can be deleted:
    modalVisible: false
}

const clothesReducer = (state = initialState, action) => {
    switch (action.type) {
        case 'LOADING_CLOTHES':
            return {
                ...state,
                clothes: [...state.clothes],
                loading: true
            }
        case 'LOAD_CLOTHES':
            return {
                ...state,
                clothes: {
                    ...state.clothes,
                    tops: {
                        category: "tops", 
                        pieces: action.payload.tops,
                        selectedPiece: 0
                    },
                    bottoms: {
                        category: "bottoms", 
                        pieces: action.payload.bottoms,
                        selectedPiece: 0
                    }
                }, 
                loading: false
            }

        case 'SELECT_NEXT_PIECE':
            let nextIndex = state.clothes[action.category].selectedPiece

            if (nextIndex === state.clothes[action.category].pieces.length - 1) {
                nextIndex = 0
            } else {
                nextIndex = state.clothes[action.category].selectedPiece + 1
            }

            return {
                ...state,
                clothes: {
                    ...state.clothes,
                    [action.category]: {
                        ...state.clothes[action.category],
                        selectedPiece: nextIndex
                    },
                }, 
                loading: false
            }

        case 'SELECT_PREVIOUS_PIECE':
            let previousIndex = state.clothes[action.category].selectedPiece

            if (previousIndex === 0) {
                previousIndex = state.clothes[action.category].pieces.length - 1
            } else {
                previousIndex = state.clothes[action.category].selectedPiece - 1
            }

            return {
                ...state,
                clothes: {
                    ...state.clothes,
                    [action.category]: {
                        ...state.clothes[action.category],
                        selectedPiece: previousIndex
                    },
                }, 
                loading: false
            }

        // These two cases can be deleted:
        case 'SHOW_MODAL':
            return {
                ...state,
                modalVisible: true
            }
        case 'HIDE_MODAL':
            return {
                ...state,
                modalVisible: false
            }

        default:
            return state
    }
}

export default clothesReducer
```


becomes:


```js
// clothesReducer.js

const initialState = { 
    clothes: [],
    loading: false
}

const clothesReducer = (state = initialState, action) => {
    switch (action.type) {
        case 'LOADING_CLOTHES':
            return {
                ...state,
                clothes: [...state.clothes],
                loading: true
            }
        case 'LOAD_CLOTHES':
            return {
                ...state,
                clothes: {
                    ...state.clothes,
                    tops: {
                        category: "tops", 
                        pieces: action.payload.tops,
                        selectedPiece: 0
                    },
                    bottoms: {
                        category: "bottoms", 
                        pieces: action.payload.bottoms,
                        selectedPiece: 0
                    }
                }, 
                loading: false
            }

        case 'SELECT_NEXT_PIECE':
            let nextIndex = state.clothes[action.category].selectedPiece

            if (nextIndex === state.clothes[action.category].pieces.length - 1) {
                nextIndex = 0
            } else {
                nextIndex = state.clothes[action.category].selectedPiece + 1
            }

            return {
                ...state,
                clothes: {
                    ...state.clothes,
                    [action.category]: {
                        ...state.clothes[action.category],
                        selectedPiece: nextIndex
                    },
                }, 
                loading: false
            }

        case 'SELECT_PREVIOUS_PIECE':
            let previousIndex = state.clothes[action.category].selectedPiece

            if (previousIndex === 0) {
                previousIndex = state.clothes[action.category].pieces.length - 1
            } else {
                previousIndex = state.clothes[action.category].selectedPiece - 1
            }

            return {
                ...state,
                clothes: {
                    ...state.clothes,
                    [action.category]: {
                        ...state.clothes[action.category],
                        selectedPiece: previousIndex
                    },
                }, 
                loading: false
            }

        default:
            return state
    }
}

export default clothesReducer
```


With these changes the action creators in `clothesActions.js` and the case logic in `clothesReducers.js` now only concern themselves with clothes and nothing else, making both files more descriptive. And instead of using Redux's central store to manage state that only one component needs to know about, that one component now manages that state locally. The app works exactly as it did before, but the code makes more sense. 