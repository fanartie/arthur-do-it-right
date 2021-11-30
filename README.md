# redux 

Let's talk about using redux with do-it-right !

We are going to create a simple redux project as following

![](redux/redux-1.gif)

The codes below looks "okay", but something is wrong!

The store

```javascript
import { createStore } from 'redux';
import reducer from "./reducer";

const store = createStore(
    reducer
);
export default store;

```


The reducer
```javascript

import initState from './initState';
import immer from 'immer';

const reducer = (state=initState, action) => {

    switch (action.type) {
        case 'add':
            return immer(state, draft =>{
                draft.list[action.idx].score += 1;
            })
        default:
            return state;
    }
}

export default reducer;
```


The initState
```javascript
module.exports = {
    list: [
        {id: 1, name: 'Tony',   score: 80},
        {id: 2, name: 'Artie',  score: 85},
        {id: 3, name: 'Joe',    score: 78},
    ]
}
```


The main code

```javascript
import store from '../../store';
import Student from '../Student';

const Main = () => {
    let list = store.getState().list;
    return (
        <div>
            {list.map((i,idx)=>{
                return (
                    <Student key={i.id} idx={idx}/>
                )
            })}
        </div>
    );
}

export default Main;

```

The Student component
```javascript
import { useDispatch, useSelector } from 'react-redux';
import { Fade } from "react-awesome-reveal";

const Student = props => {

    const dispatch = useDispatch();

    let list = useSelector(state => state.list);
    let one = list[props.idx];

    console.log('update', one.name);

    const onClick = () => {
        dispatch({
            type: 'add',
            idx: props.idx
        });
    }

    let key = [one.id, one.score].map(String).join(':');

    return (
        <Fade key={key}>
            <div>
                {one.id}, {one.name}, {one.score},
                <button onClick={onClick}>add</button>
            </div>
        </Fade>
    );
}

export default Student;

```
