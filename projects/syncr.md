# Syncr

A simple Javascript package to make `asynchronous` requests to a server over WebSocket/Http connection

[View Source](https://github.com/CERP/syncr)

## Install

Install with npm or yarn via

```
yarn add @cerp/syncr@1.1.2
```

or

```
npm i @cerp/syncr@1.1.2
```

## API

```ts
type Event = "connect" | "disconnect" | "message" | "verify"
class Syncr {
    url: string
    ready: boolean
    ws?: WebSocket
    pingInterval?: number
    pending: Map<string, {
        resolve: (a: any) => any
        reject: (a: any) => any
    }>
    message_timeout: number
    connection_verified: boolean
    private onEventFunctions
    private onNextEventFunctions
    constructor(url: string)
    verify(): void
    connect(): Promise<void>
    on(event: Event, f: Function): void
    onNext(event: Event, f: Function): void
    private trigger
    cleanup(): void
    ping(): void
    send(message: any, timeout?: number): Promise<any>
}
```

## Usage

- use `syncr` while initializing the redux store

  ```ts
  import Syncr from '@cerp/syncr'
  import { applyMiddleware, AnyAction, createStore, Store } from 'redux'

  const syncr = new Syncr("wss://example.com/ws")
  syncr.on('connect', () => store.dispatch(connected()))

  const store: Store<RootState> = createStore(reducer, initial_state, applyMiddleware(thunkMiddleware.withExtraArgument(syncr) as ThunkMiddleware<RootState, AnyAction, Syncr>))
  ```

- make request to server using `Syncr.Send()` with `Promise`

  ```ts
  import Syncr from '@cerp/syncr'
  export const startService = () => {

    const syncr = new Syncr("wss://example.com/ws")
    syncr.send({
      type: "START_SERVICE",
      client_type: "CERP"
      payload: {
        initiator_id: "xxxx-xxxx"
        req_token: "xxxx-xxxx"
        req_timeout: 5000
      }
    })
      .then(response => {
        // do something here
      })
      .catch(error => {
        // do something here
      })
  }
  ```

  `OR with async arrow style`

  ```ts
  import Syncr from '@cerp/syncr'
  export const startService = aysnc () => {

    const syncr = new Syncr("wss://example.com/ws")
    try {
      const response = await syncr.send({
        type: "START_SERVICE",
        payload: {
          initiator_id: "xxxx-xxxx"
          req_token: "xxxx-xxxx"
          req_timeout: 5000
        }
      })
  
      // do something with reponse here

    } catch(error) {
      // do something with error here
    }
  }
  ```