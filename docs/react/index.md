<!--
 * @Author: pony@diynova.com
 * @Date: 2021-12-15 00:18:01
 * @LastEditors: pony@diynova.com
 * @LastEditTime: 2021-12-15 00:20:09
 * @FilePath: /notes/docs/react/index.md
 * @Description: redux 原理
-->
# [Redux 原理](http://cn.redux.js.org/)

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>redux 原理</title>
    </head>
    <body>
        <div>
            <button type="button" onclick="store.dispatch({type: 'sub', n : 2})">-</button>
            <span id="coundDisplay">10</span>
            <button onclick="store.dispatch({type: 'add', n : 3})">+</button>
        </div>
        <script type="text/javascript">
            const countDisplay = document.querySelector("#coundDisplay")
            const counterState = {
                count: 5
            }

            const reducer = (state, action) => {
                if(!state) {
                    return counterState;
                }
                switch (action.type) {
                    case 'add':
                        return {
                            ...state,
                            count: state.count += action.n
                        }
                        break;
                    case 'sub':
                        return {
                            ...state,
                            count: state.count -= action.n
                        }
                        break;
                    default:
                        return state;
                }
            }

            const createStore = (reducer) => {
                let state = null
                const getState = () => state
                const listerners = []

                const subscribe = (listener) => listerners.push(listener)

                const dispatch = (action) => {
                    state = reducer(state, action)
                    listerners.forEach(listener => listener())
                }
                dispatch({})
                return {
                    getState,
                    dispatch,
                    subscribe
                }
            }

            const store = createStore(reducer)
            const renderCount = () => {
                countDisplay.innerHTML = store.getState().count
            }
            renderCount()
            store.subscribe(renderCount)

        </script>
    </body>
</html>
```