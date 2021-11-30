# redux 

Let's talk about common mistakes with Redux

We are going to create a simple Redux project as following

![](redux-1.gif)

### _The codes below look good, but something are wrong!_



The store

```jsx
import { createStore } from 'redux';
import reducer from "./reducer";

const store = createStore(
    reducer
);
export default store;

```


The initState
```jsx
module.exports = {
    list: [
        {id: 1, name: 'Tony',   score: 80},
        {id: 2, name: 'Artie',  score: 85},
        {id: 3, name: 'Joe',    score: 78},
    ]
}
```

The reducer
```jsx

import initState from './initState';

const reducer = (state=initState, action) => {

    switch (action.type) {
        case 'add':
            let newList = JSON.parse(JSON.stringify(state.list));
            newList[action.idx].score += 1;
            return {
                ...state,
                list: newList
            };
        default:
            return state;
    }
}

export default reducer;
```





The main code

```jsx
import {useSelector} from "react-redux";
import Student from '../Student';

const Main = () => {

    let list = useSelector(state => state.list);

    console.log('update Main');

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
```jsx
import { useDispatch, useSelector } from 'react-redux';
import { Fade } from "react-awesome-reveal";

const Student = props => {

    const dispatch = useDispatch();

    let list = useSelector(state => state.list);
    let person = list[props.idx];

    console.log('update', person.name);

    const onClick = () => {
        dispatch({
            type: 'add',
            idx: props.idx
        });
    }

    let key = [person.id, person.score].map(String).join(':');

    return (
        <Fade key={key}>
            <div>
                {person.id}, {person.name}, {person.score},
                <button onClick={onClick}>add</button>
            </div>
        </Fade>
    );
}

export default Student;


```

##### _What's wrong ?_

