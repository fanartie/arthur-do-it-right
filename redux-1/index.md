# Redux - avoid rendering everything

by [Arthur Fan](mailto:fanartie@gmail.com),  July 2020

Let’s talk about common mistakes with using Redux.
We are going to create a simple Redux project as follows.

關於 Redux 經常遇到的問題, 多半都是 "看不見的問題", 因為顯示的結果對了 以為就沒事了, 事實上, 看不到的錯誤非常多, 例如 "重複 Render"

![](redux-1.gif)

### _The code below looks good but needs to be corrected!_

以下的基本的 Redux 範例, 非常簡單, 執行也正確, 但事實上有 bug... 

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

    console.log('update Main ===');

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

### _What's wrong with the code?_

到底哪裡出錯了, 我們來看這個動畫, 當一個學生被按下, 我們從 console.log 證明, 其實 所有人 都被 render 了 !

(Test-1)

![](redux-2.gif)

### _We should NOT reload the whole "Main" when clicking on a single person!_

如果只有一個學生被按下, 我們不應該 reload 整個 Main component 

How to fix it? If we want to reference the initState "list" at the "Main".

Simply use "store.getState()" to load the data.

We should avoid using the "Main(root)" as an observer because any update to the root may force rendering all the children.

我們應該避免 在 parent 層 進行 render 觸發, 因為這樣 所有的 child 都會被重新 rendered

```jsx
import store from '../../store';
import Student from '../Student';

const Main = () => {

    let list = store.getState().list;

    console.log('update Main ===');

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

### _We have fixed the issue. The "Main" is only rendering once now!_
### _But why it still renders all the students when we click on a single person?_

我們已經證明 main 只有 render 一次, 但是 儘管只有一人被按下, 為什麼 仍然 所有學生 都被重複 render 呢 ? 

(Test-2)

![](redux-3.gif)

We understood it was attempting to overwrite an immutable state, and it used a quick-dirty solution as "JSON.parse(JSON.stringify())", which allocates new memory for the value and then every array element will be considered as 'updated'.  

因為這裡 用了一個 偷懶的辦法, 雖然只有改一個人, 事實上 我們把整個 array 都重置了, 然而, 為什麼我們要用這個奇怪的方法 JSON.parse(JSON.stringify()) 呢 ? 因為 state 變數是 immutable 無法更改, 我們就偷懶 整個取代掉, 難怪 所有學生 都被強迫 render.  

How can we reduce the immutable state by updating only one array element?

我們如何做到, 在 immutable array, 僅更改其中一個 element value 呢 ? 

我經常用 immer 這個 library.

Introducing the 'immer' library.
[https://www.npmjs.com/package/immer](https://www.npmjs.com/package/immer)

```bash
yarn add immer
```

The reducer with using 'immer'

```jsx
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

### _Awesome! But it doesn't fix the issue._

為什麼 用了 immer 僅改了其中一個元素, 問題還是沒解決 ?

Because the useSelector() is subscribing entire array, since only one student is updated, it will still consider the whole array is updated. 

因為 useSelector() 訂閱了整個 array, 即使是只有一個 element 被更改, 仍然被視為 整個 Array 內容都要 render.

(Test-3)

![](redux-4.png)

Each student should only subscribe to their element instead of the entire array.

我們必須讓 useSelector 精準到 訂閱 每一個元素 !!! 

```jsx
let person = useSelector(state => state.list[props.idx]);
```

The new code...
```jsx
import { useDispatch, useSelector } from 'react-redux';
import { Fade } from "react-awesome-reveal";

const Student = props => {

    const dispatch = useDispatch();

    let person = useSelector(state => state.list[props.idx]);

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

(Test-4)

![](redux-5.gif)

### _Finally, it's running correctly now!_

太棒了, 終於正常 render 了 !!! 有多少看不到的地雷啊 ?

#### _Questions for you..._

When you click on a person, you might have noticed the "react-awesome-reveal" used for the "Fade" effect. Please look carefully at the (Test-1). Why is it only "Fading" on one person since we have proved that all students are rendered?


請試著回答 這個思考題 非常有意思的喔 !

你可以在 (Test-1) 動畫看到, 我們有使用 "Fade" 效果, 既然我們都已經證明, 所有學生都被 rendered 了, 那為什麼僅有一人 會出現 Fade 效果呢 ?



