---
author: Mudassar
title: An Overview of Labs project
---

# Intro to LABS projects

In this markdown, you will learn about the technologies and architecture of some of `Labs` projects. You can find all the projects at [CERP Github](https://github.com/cerp) organization.

## Technologies

We're building system using latest technologies, here's the list but we're not limited to them:

- **Frontend**

  - Javascript
  - Typescript
  - ReactJS + Redux
  - MaterialUI

- **Backend**

  - Elixir
  - Postgres

- **Platform**

  - Google Cloud Platform
  - Docker
  - Kubernetes

## Tool we built

Here are some tools that we build which are currently used in most of our projects

- **Syncr**

  Make `asynchronous` requests to a server over WebSocket/Http connection

  [Learn More](./syncr.md)

- **Former**

  Handle `HTMLFormEvents` and mutate the state. Currently support `class` based components

  [Learn More](./former.md)
  
- **Dynamic**

  Provide useful tooling to manipulate the complex `Objects` (JS) and `Maps` (Elixir)

  [Learn More](./dynamic.md)

## Projects overview

At `Labs` we built [`MISchool`](https://mischool.pk) and [`IlmxExchange`](https://ilmexchange) as our starter products. Below are some stuff which we follow and reuse for every project that we already built and will building in future.

### **Actions**

To make change to `redux store` or to send payload to server, we divide actions into two category

- **actions**

  These actions can be futher categorized into `Core` and `Simple` (based on core actions) actions. Let's see how core actions works

  **Core Actions**
  
  For now we have `createMerges()` and `createDeletes()` actions which help to manipulate to complex deep object in `state`

  - `createMerges()`

    createMerges() takes single argument of type `Merge[]` and return `void`. Here we're only looking at the prototype and usage of createMerges().

    **API**

    ```ts
    type Merge = {
      path: Array<string>
      value: any
    }
    createMerges(merges: Merge[]): void
    ```

    ````txt
    What does merge mean?

    Basically merge is an object which contains path and value. "Path" is composed of complex deep object properties
    and "value" is any thing that we want to to store or update against the path.
    ````

    **Usage**

    Let's see createMerges in actions. Consider we have state like below

    `State A`

    ```ts
    "db": {
      "students": {
        [1]: {
          id: 1
          Name: "ABC",
          Gender: "M",
         "attendance": {}
        }
      }
    }
    ```

    1st Merge: Let's add a single student entry to state, so we create a merge like

    ```ts
    // merge to create or update a student
    const student = {
      id: 2
      Name: "XYZ",
      Gender: "X",
      payments: {}
    }

    const merge = [
      {
        path: ["db", "students", student.id],
        value: student
      }
    ]

    dispatch(createMerges(merge))
    ```

    After the merge action, state changes from `A` => `B` with a new student of `id:2`

    `State B`

    ```ts
    "db": {
      "students": {
        [1]: {
          id: 1
          Name: "ABC",
          Gender: "M",
         "attendance": {}
        },
        [2]: {
          id: 2
          Name: "XYZ",
          Gender: "M",
         "attendance": {}
        }
      }
    }
    ```

    2nd Merge: Let's add an `attendance` entry against a student (deep object), so we create a merge like

    ```ts
    // merge to create or update a student
    const student_id = 2
    const attendance = {
      date: "04-07-2020",
      status: "PRESENT",
      time: 1593806556000
    }

    const merge = [
      {
        path: ["db", "students", student_id, "attendance", attendance.date],
        value: attendance
      }
    ]

    dispatch(createMerges(merge))
    ```

    After the 2nd merge action, state changes from `B` => `C` with an attendance entry of student `id:2`

    `State C`

    ```ts
    "db": {
      "students": {
        [1]: {
          id: 1
          Name: "ABC",
          Gender: "M",
         "attendance": {}
        },
        [2]: {
          id: 2
          Name: "XYZ",
          Gender: "M",
         "attendance": {
            date: "04-07-2020",
            status: "PRESENT",
            time: 1593806556000
          }
        }
      }
    }
    ```
  
  - `createDeletes()`

    createDeletes() takes single argument of type `Delete[]` and return `void`. Here we're only looking at the prototype and usage of createDeletes().

    **API**

    ```ts
    type Delete = {
      path: Array<string>
    }
    createDeletes(deletes: Delete[]): void
    ```

    **Usage**

    1st Delete: Let's say we want to delete a student of `id: 1`, so we create a delete like

    ```ts
    const student_id = 1
    const delete = [
      {
        path: ["db", "students", student_id]
      }
    ]
    dispatch(createDeletes(delete))
    ```

    After a `createDeletes()` action, state changes from `C` => `D`, deleted student with `id:1`

    `State D`

    ```ts
      "db": {
        "students": {
          [2]: {
            id: 2
            Name: "XYZ",
            Gender: "M",
          "attendance": {
              date: "04-07-2020",
              status: "PRESENT",
              time: 1593806556000
            }
          }
        }
      }
    ```

- **`async` actions**

  `async` actions help to send data to server using `Syncr`.

  How a state manipulation by createMerges() or createDeletes encoded as an action and send to server. Let's see with createDeletes() in below steps. Assuming that we're working on `MISchool`

  - `Step 1`: Dispatch createDeletes()
  
    ```ts
    const student_id = 1
    const delete = [
      {
        path: ["db", "students", student_id]
      }
    ]
    dispatch(createDeletes(delete))
    ```

  - `Step-2`: Prepare merges as an action for server (in `createDeletes()`)

    ```ts
      const new_deletes = deletes.reduce((agg, curr) => ({
        ...agg,
        [curr.path.join(',')]: {
          action: {
            type: "DELETE",
            path: curr.path.map(p => p === undefined ? "" : p),
            value: 1 // what's the reason behind this?
          },
          date: new Date().getTime()
        }
      }), {})
  
      const state = getState()
      const rationalized_deletes = getRationalizedQueuePayload(new_deletes, "mutations", state)
    ```

  - `Step-3`: Rationalize `state.queued` items with new delete mutations (in `getRationalizedQueuePayload()`)
  
    In this step, we're making sure, if there's anything else in queue(not send to server before), merge them together
    and send to server if `state.connected === true`
  
    ```ts
    // we're mainly dealing with three types of payload, deletes, merges fall in "mutations"
    type QueuedItem = "mutations" | "images" | "analytics"

    const getRationalizedQueuePayload = (payload: any, key: QueuedItem, state: RootReducerState["queued"]) => {
      return {
        ...state.queued,
        images: {},
        [key]: {
          ...state.queued[key],
          ...payload
        }
      }
    }
    ```

    Here's what `RootReducerState` looks like

    ![root reducer state](./root-reducer-state.png)

  - `Step-4`: Send payload to server using `Syncr`

    ```ts
      const state = getState()
      syncr.send({
        type: SYNC,
        school_id: state.auth.school_id,
        client_type: client_type,
        lastSnapshot: state.lastSnapshot,
        payload
      })
    ```

## Code Formatting + Linting

For code formatting and liting we use `ESLint` for Typescript
