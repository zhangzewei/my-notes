# Skills of writing unit test for react
> Author: markzzw | Date: 2023-12-30

> code repo: https://github.com/zhangzewei/Skills-of-writing-unit-test-for-react-demo-code


After writing some unit tests for react project, I summarised some skills of writing unit tests for react.

## Tech stack
This is the tech stack of it, we need to prepare it.

1. vite - react (react 18)
2. vitest (1.0.4)
3. @testing-library/react (14.1.2)

## Setup test env
Before we start it, we need to set up the test env.

### 1. install dependencies
```shell
npm install vitest
npm install @testing-library/jest-dom
npm install @testing-library/react
```
### 2. Add configuration in vite-config
```ts
/// <reference types="vitest" />
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
    css: true,
    include: ['src/**/*.test.[jt]s?(x)'],
  },
  envDir: 'env',
})
```
### 3. Create `./src/test/setup.ts`
```ts
import '@testing-library/jest-dom';

vi.stubGlobal('globalEventYonWantToMock', vi.fn());
```

### 4. Set script in package.json
```json
{
    "scripts": {
        "test": "vitest",
    }
}
```
Then we can run the script `npm run test` in the console.
## Test render with different props
In this kind of Component, use `render()` to render different props, and test the component render if is correct.
```jsx
import { render } from "@testing-library/react";
import Component from "./index";

describe('Component', () => {
    it('should render correctly when set prop1', () => {
        const { queryByTestId } = render(
            <Component prop='prop1' />
        );
        expect(queryByTestId('testid1')).toBeTruthy();
    });
    it('should render correctly when set prop2', () => {
        const { queryByTestId } = render(
            <Component prop='prop2' />
        );
        expect(queryByTestId('testid2')).toBeTruthy();
    });
})
```
## Test with HTML_EVENTs
When talking about unit tests, we can't avoid the event test, there are two ways to write it, and we need to use `act(() => {})` to wrap the event step.
### Trigger event directly
This way is always used to test the `click` events.
```jsx
import { render, act } from "@testing-library/react";
import Component from "./index";

describe('Component', () => {
    it('should do something when click xxx button', () => {
        const { queryByTestId } = render(
            <Component />
        );
        const buttonEle = queryByTestId('button');
        act(() => {
            buttonEle.click();
        });
        expect(/* some result after click button */).toBe();
    });
})
```
Also if `Component` needs to pass the function through the props, we can test the function if be called, we can use `vi.fn()` as the mock spy.
```jsx
import { render, act } from "@testing-library/react";
import Component from "./index";

describe('Component', () => {
    it('should do something when click xxx button', () => {
        const spy = vi.fn();
        const { queryByTestId } = render(
            <Component onClick={spy} />
        );
        const buttonEle = queryByTestId('button');
        act(() => {
            buttonEle.click();
        });
        expect(spy).toHaveBeenCalledTimes(1);
    });
})
```
### Trigger event by using `fireEvent` provided by the `@testing-library/react`
This way is mostly used on `input event`, `scroll event`, and some events that need to pass/get value.
```jsx
import { render, act } from "@testing-library/react";
import Component from "./index";

describe('Component', () => {
    // change event
    it('should do something when trigger some event, () => {
        const spy = vi.fn();
        const { queryByTestId } = render(
            <Component onChange={spy} />
        );
        const inputEle = queryByTestId('input');
        act(() => {
            fireEvent.change(inputEle, {target: {value: '23'}});
        });
        expect(spy).toHaveBeenCalledTimes(1);
    });
    // keyDown event
    it('should do something when trigger some event, () => {
        const spy = vi.fn();
        const { queryByTestId } = render(
            <Component onChange={spy} />
        );
        const inputEle = queryByTestId('input');
        act(() => {
            fireEvent.keyDown(inputEle, {
                target: {value: '23'},
                code: 'Enter'
            });
        });
        expect(spy).toHaveBeenCalledTimes(1);
    });
});
```
If your event function has been wrapped by a setTimeOut function like `debounce/throttle`, then we need to use `async/await` and `waitFor(() => {})` to help us.
```jsx

import { render, act, waitFor } from "@testing-library/react";
import Component from "./index";

describe('Component', () => {
    // usually we will use debounce to wrap a change function
    it('should do something when trigger some event, async () => {
        const spy = vi.fn();
        const { queryByTestId } = render(
            <Component onChangeWrappedByDebounce={spy} />
        );
        const inputEle = queryByTestId('input');
        act(() => {
            fireEvent.change(inputEle, {target: {value: '23'}});
        });
        // due to use the debounce
        await waitFor(() => expect(spy).toHaveBeenCalledTimes(1));
    });
});
```
## Test hook function
Usually, we will write hooks to help us, so we need to test the hook function too, for hook we need to use `renderHook()` to help us.

`renderHook` also is provided by `@testing-library/react`.

Here is a hook function that I write to get detail data, I use the `useEffect` to update the detail when I change the dependencies, and I use a loading status from the redux store.

This is a typical fetch data hook, let's test it.

**useDetailData.ts**
```js
import { useEffect, useState } from "react";
import { useDispatch } from "react-redux";
import { IReuqestProps } from "./request";
import { updateLoading } from "./loadingStore";

export default function useDetailData<DETAIL_TYPE>(
    getDataRequest: (...args: any[]) => Promise<IReuqestProps<DETAIL_TYPE>> | undefined,
    dependencies: unknown[],
) {
    const dispatch = useDispatch();
    const [detail, setDetail] = useState<DETAIL_TYPE | null>(null);
    useEffect(() => {
        const shouldGetDetail = dependencies.find(d => !!d);
        if (!!shouldGetDetail) {
            dispatch(updateLoading(true));
            getDataRequest(...dependencies)?.then(res => {
                res.data && setDetail(res.data);
            }).finally(() => {
                dispatch(updateLoading(false));
            });
        } else {
            setDetail(null);
        }
    }, dependencies);

    return detail;
}
```
**useDetailData.test.tsx**
First I write a function to mock a fetch function, because we use the `Promise` the setTtimeOut function, so we need to use `async/await`.

As I noticed I use a loading status from redux store, so we also need to mock the redux, so I write the `ReduxProviderWrapper`.

All right, all the setups are ready, now we use the `renderHook()` to render our hook, the first param of the `renderHook` is a function that we can set the props, the props are what the hook function params have been, and we can set initialProps in the second param of the `renderHook`, so that we can use the `rerender(newProps)` returned by `renderHook` to change dependencies, and test the result if is correct, and don't forget to set the wrapper `ReduxProviderWrapper` in the second param of `renderHook` when we use the redux.
```jsx
import { renderHook, waitFor } from '@testing-library/react'
import useDetailData from './useDetailData'
import { ReduxProviderWrapper } from '../../test/utils'
import store from "./store";

const ReduxProviderWrapper = ({ children }: PropsWithChildren) => (
    <Provider store={store}>{children}</Provider>
);

describe('useDetailData', () => {
    let getMockPromise: any;
    beforeEach(() => {
        getMockPromise = (data: any) => {
            return function() {
                return Promise.resolve({
                    code: 200,
                    data,
                    message: 'success',
                })
            }
        }
    })
    it('should return the data with the succeed http request', async () => {
        const { result, rerender } = renderHook(
            (props) => useDetailData(props.promise, props.dependencies),
            { 
                wrapper: ReduxProviderWrapper,
                initialProps: {
                    promise: getMockPromise('data'),
                    dependencies: ['test dependency']
                }
            }
        );
        await waitFor(() => {
            expect(result.current).toBe('data');
        });
        rerender({promise: getMockPromise('data2'), dependencies: ['test dependency changed'] });
        await waitFor(() => {
            expect(result.current).toBe('data2');
        });
    });
});
```
Now we finished most tests of this hook, but if we need to get our coverage to be 100% we still need to test one if we don't set any dependency, we just need to add one more test case.
```jsx
it('should return the null without any dependency', async () => {
    const { result } = renderHook(
        () => useDetailData(getMockPromise('data'), []),
        { wrapper: ReduxProviderWrapper }
    );
    await waitFor(() => {
        expect(result.current).toBe(null);
    });
});
```
Then we can get our satisfactory test coverage report.
**Test coverage**
File|% Stmts|% Branch|% Funcs|% Lines|Uncovered Line 
-|-|-|-|-|-          
useDetailData.ts|100|100|100|100|     

## Test with react-router
If we use the `react-router`, we also need to provide the router context when we test it.

How to provide router context? Using `MemoryRouter` which is provided by react-router.

For example, I test a hook using the router hook `useLocation()`, so I need to provide a router context as a wrapper for `renderHook()`.

**useQuery hook**
```jsx
import React from "react";
import { useLocation } from "react-router-dom";

export default function useQuery() {
    const { search } = useLocation();
    return React.useMemo(() => new URLSearchParams(search), [search]);
}
```
**useQuery.test.tsx**
Through the `initialEntries` props to mock the route currently used.

```jsx
import { render } from "@testing-library/react";
import useQuery from "./useQuery";
import { RouterProviderWrapper } from "../../test/utils";

const RouterProviderWrapper = (props: MemoryRouterProps) => (
    <MemoryRouter {..._omit(props, 'children')}>
        {props.children}
    </MemoryRouter>
);

describe("useQuery", () => {
    it('should return the query when there is a query string on the url.', () => {
        const Test = () => {
            const query = useQuery().get('value');
            return <div data-testid="test">{query}</div>;
        }
        const { queryByTestId } = render(
            <RouterProviderWrapper initialEntries={['/test-url?value=testValue']}>
                <Test />
            </RouterProviderWrapper>
        );
        const ele = queryByTestId('test');
        expect(ele).toHaveTextContent('testValue');
    });
})
```
## Test with the mock function
Sometimes we should mock some functions to return the result as we need so that we can test the function's behavior if is correct.

For this `vitest` provide the `vi.mock('path/of/import/function')` & `vi.mocked().mockReturnValue()`.

**usePosts.ts**
In this hook, the request function `getPost` is a fetch function imported from the request file, but when we test it we don't want to get the real data back, we need to use mock data so that we can control the `expct` statement.
```js
import { useState, useEffect } from "react";
import { getPosts } from "./request";
import { Post } from "./dto.ts";

export default function usePosts (query: string) {
    const [posts, setPosts] = useState<Post[]>([]);
    const [loading, setLoading] = useState(false);
    const handleGetPosts = () => {
        setLoading(true);
        query && getPosts(query).then(res => {
            if (res.data && res.data.length) {
                setPosts(res.data);
            }
        }).finally(() => setLoading(false));
    }
    useEffect(() => {
        handleGetPosts();
    }, [query]);
    return { posts, loading };
}
```
**usePost.test.ts**
First use `vi.mock()` to mock the import function, this should do outside the `describe`, this is a requirement from `vitest`.

Then use `vi.mocked(importFunction).mockReturnValue(mockValue)` to mock the function and its return value.

Also should use `async/await` & `waitFor` because of the promise fetching.
```js
import { renderHook, waitFor } from "@testing-library/react";
import usePost from "./usePost";
import { getPost } from "./request";

vi.mock('./request', () => {
    return {
        getPost: vi.fn(),
    }
});

describe('usePost', () => {
    vi.mocked(getPost).mockReturnValue(
        Promise.resolve(
            {
                data: [
                    {
                        id: 'testid',
                        title: 'title',
                        content: 'content'
                    }
                ]
            }
        )
    )
    it('should return the data with the succeed http request', async () => {
        const { result } = renderHook(
            () => usePost('query'),
        );
        expect(result.current.loading).toBe(true);
        expect(result.current.post.length).toBe(0);
        await waitFor(() => {
            expect(result.current.loading).toBe(false);
            expect(result.current.post.length).toBe(1);
        });
    });
});
```

## Summary
At the end of this document, we have learned so many skills of writing unit tests for react, I hope those skills can help you to write unit tests easier.

Thx for reading.