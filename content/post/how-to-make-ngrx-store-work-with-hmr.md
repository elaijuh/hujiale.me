+++
categories = ["tech"]
date = "2016-09-11T03:26:53+08:00"
draft = false
slug = ""
tags = ["angular2", "ngrx"]
title = "How to make ngrx/store work with HMR"

+++
In my previous post, I talked of a way to develop angular 2 app with HMR.
The vendors I use are *@angularclass/hmr* and *@angularclass/hmr-loader*
Later on, I thought I might need a data flow tool like redux to manage my app state and I found [ngrx/store](https://github.com/ngrx/store)

@angularclass/hmr injects some *hmr* prefix life cycles into the main module to let you to restore the data.
But app state management is optional and you can choose your own way to implement it, so I will walk you through how I implement HMR with ngrx/store

### Retrieve the current state

To retrive the current app state before it's deposed is easy, just subscribe it:

```js
this._store.take(1).subscribe(s => store.rootState = s)
```

That's it

### Restore the current state

This is kinda cumbesome as `ngrx/store` doesn't provide a way to set the root state, I need to compose a rootReducer to do this.
With the help of [Mike Ryan](https://github.com/MikeRyan52), I figure out a way to do that.

```js
function stateSetter(reducer: ActionReducer<any>): ActionReducer<any> {
  return function (state, action ) {
    if (action.type === 'SET_ROOT_STATE') {
      return action.payload
    }
    return reducer(state, action)
  }
}

const rootReducer = compose(stateSetter, combineReducers)({
    // your reducers here
})
```

Now I can dispatch a `SET_ROOT_STATE` action to reset the app state to what I have stored

### Get everything together

This is very much based on [angular2-seed](https://github.com/mgechev/angular2-seed), you might have your own `main.ts` though

```js
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic'
import { BrowserModule } from '@angular/platform-browser'
import { RouterModule } from '@angular/router'
import { NgModule, ApplicationRef } from '@angular/core'
import { removeNgStyles, createNewHosts, createInputTransfer, bootloader } from '@angularclass/hmr'

import { compose} from '@ngrx/core/compose'
import { Store, StoreModule, ActionReducer, combineReducers } from '@ngrx/store'
import { StoreDevtoolsModule } from '@ngrx/store-devtools'
import { StoreLogMonitorModule, useLogMonitor } from '@ngrx/store-log-monitor'

import { AppModule } from './app'
import { AppComponent } from './app/app.component'
import { message } from './reducer'


// Generate a reducer to set the root state
function stateSetter(reducer: ActionReducer<any>): ActionReducer<any> {
  return function (state, action ) {
    if (action.type === 'SET_ROOT_STATE') {
      return action.payload
    }
    return reducer(state, action)
  }
}

const rootReducer = compose(stateSetter, combineReducers)({
  message
})

let imports = [
  BrowserModule,
  RouterModule.forRoot([], {
    useHash: true
  }),
  // app
  AppModule,
  // vendors
  StoreModule.provideStore(rootReducer)
]

// Enable HMR and ngrx/devtools in hot reload mode
if (module.hot) imports.push(...[
    StoreDevtoolsModule.instrumentStore({
      monitor: useLogMonitor({
        visible: true,
        position: 'right'
      })
    }),
    StoreLogMonitorModule,
])

@NgModule({
  bootstrap: [ AppComponent ],
  declarations: [ AppComponent ],
  imports
})
class MainModule {
  constructor(public appRef: ApplicationRef, private _store: Store<any> ) {}
  hmrOnInit(store) {
    if (!store || !store.rootState) return

    // restore state by dispatch a SET_ROOT_STATE action
    if (store.rootState) {
      this._store.dispatch({
        type: 'SET_ROOT_STATE',
        payload: store.rootState
      })
    }

    if ('restoreInputValues' in store) { store.restoreInputValues() }
    this.appRef.tick()
    Object.keys(store).forEach(prop => delete store[prop])
  }
  hmrOnDestroy(store) {
    const cmpLocation = this.appRef.components.map(cmp => cmp.location.nativeElement)
    this._store.take(1).subscribe(s => store.rootState = s)
    store.disposeOldHosts = createNewHosts(cmpLocation)
    store.restoreInputValues  = createInputTransfer()
    removeNgStyles()
  }
  hmrAfterDestroy(store) {
    store.disposeOldHosts()
    delete store.disposeOldHosts
  }
}

export function main() {
  return platformBrowserDynamic().bootstrapModule(MainModule)
}

bootloader(main)
```

Now the state remains the same after you change your code with HMR on, cool right.


