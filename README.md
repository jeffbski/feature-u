# feature-u

TODO: once words are finalized, **retrofit all links**

**feature-u** is a utility library that _facilitates feature-based
project organization_ in your {{book.ext.react}} project.  It assists
in organizing your project by individual features.

Most software engineers would agree that organizing your project by
feature is much preferred over type-based patterns.  Because
application domains grow in the real world, project organization by
type simply doesn't scale, it just becomes unmanageable!  There are a
number of good articles that discuss this topic _(with insights on
feature-based design and structure)_.

**feature-u** is a utility library that manages and streamlines this
process.  It automates some of the mundane details of your features
and helps in promoting features that are **plug-and-play**.

The following article is an introduction to **feature-u**, with
examples from a real-world app: {{book.ext.eateryNod}} _(where
**feature-u** was conceived)_: `TODO: Introducing: feature-u (Feature
Based Project Organization for React)`

**feature-u** allows you to **focus your attention on the "business
end" of your features!**

TODO: Badges Here
<!--- Badges for CI Builds
?? retrofit this from action-u
[![Build Status](https://travis-ci.org/KevinAst/action-u.svg?branch=master)](https://travis-ci.org/KevinAst/action-u)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/ab82e305bb24440281337ca3a1a732c0)](https://www.codacy.com/app/KevinAst/action-u?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=KevinAst/action-u&amp;utm_campaign=Badge_Grade)
[![Codacy Badge](https://api.codacy.com/project/badge/Coverage/ab82e305bb24440281337ca3a1a732c0)](https://www.codacy.com/app/KevinAst/action-u?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=KevinAst/action-u&amp;utm_campaign=Badge_Coverage)
[![Known Vulnerabilities](https://snyk.io/test/github/kevinast/action-u/badge.svg)](https://snyk.io/test/github/kevinast/action-u)
[![NPM Version Badge](https://img.shields.io/npm/v/action-u.svg)](https://www.npmjs.com/package/action-u)
---> 


## Install

```shell
npm install --save feature-u
```

## feature-u Basics

The basic process of **feature-u** is that each feature promotes a
{{book.api.Feature}} object that contains various aspects of that
feature ... _things like: the feature's name, it's Public API, whether
it is enabled, initialization constructs, and resources used to
configure it's slice of the frameworks in use._

In turn, these {{book.api.Feature}} objects are supplied to
{{book.api.launchApp}}, which configures and starts your application,
returning an {{book.api.App}} object (_which promotes the public API
of each feature_).

In **feature-u**, "aspect" is a generalized term used to refer to the
various ingredients that (when combined) constitute your application.
Aspects can take on many different forms: **UI Components** and **Routes**
&bull; **State Management** _(actions, reducers, selectors)_ &bull;
**Business Logic** &bull; **Startup Initialization Code** &bull;
_etc. etc. etc._

A fundamental goal of **feature-u** is to **automatically configure
the framework(s)** used in your run-time-stack _(by accumulating the
necessary resources across all your features)_.  Because not everyone
uses the same frameworks, **feature-u** accomplishes this through
**Extendable Aspects** _(you can find them in external NPM packages,
or you can create your own)_.  The interface to your chosen frameworks
is not altered in any way.  You use them the same way you always have
_(just within your feature boundary)_.


## Usage

The basic usage pattern of **feature-u** is to:

1. Choose the Aspects that you will need, based on your selected
   frameworks (i.e. your run-time stack).  This extends the aspect
   properties accepted by the Feature object (for example:
   `Feature.reducer` for {{book.ext.redux}}, or `Feature.logic` for
   {{book.ext.reduxLogic}}).

   Typically these Aspects are packaged separately in NPM, although you
   can create your own Aspects (if needed).

1. Organize your app into features.

   * Each feature should be located in it's own directory.

   * How you break your app up into features will take some time and
     thought.  There are many ways to approach this from a design
     perspective.

   * Each feature will promote it's aspect content through a Feature
     object (using {{book.api.createFeature}}).

1. Your mainline starts the app by invoking {{book.api.launchApp}},
   passing all Aspects and Features.

**Easy Peasy!!**


## Directory Structure

Here is a sample directory structure of an app that uses **feature-u**:

```
src/
  app.js              ... launches app using launchApp()

  feature/
    index.js          ... accumulate/promote all app Feature objects

    featureA/         ... an app feature
      actions.js
      appDidStart.js
      appWillStart.js
      comp/
        ScreenA1.js
        ScreenA2.js
      index.js        ... promotes featureA object using createFeature()
      logic.js
      reducer.js
      route.js

    featureB/         ... another app feature
      ...

  util/               ... common utilities used across all features
    ...
```

Each feature is located in it's own directory, containing it's aspects
(actions, reducers, components, routes, logic, etc.).

## Feature Object

Each feature promotes it's aspect content through a
{{book.api.Feature}} object (using {{book.api.createFeature}}).

**`src/feature/featureA/index.js`**
```js
import {createFeature}  from 'feature-u';
import reducer          from './state';
import logic            from './logic';
import route            from './route';
import appWillStart     from './appWillStart';
import appDidStart      from './appDidStart';

export default createFeature({
  name:     'featureA',
  enabled:  true,

  publicFace: {
    api: {
      open:  () => ... implementation omitted,
      close: () => ... implementation omitted,
    },
  },

  reducer,
  logic,
  route,

  appWillStart,
  appDidStart,
});
```

The docs will fill in more detail, but for now notice that the feature
is conveying reducers, logic modules, routes, and does some type of
initialization (appWillStart/appDidStart).  It also promotes a
publicFace (open/close) that can be used by other features (i.e. the
feature's Public API).


## launchApp()

In **feature-u** the application mainline is very simple and generic.
There is no real app-specific code in it ... **not even any global
initialization**!  That is because **each feature can inject their own
app-specific constructs**!!  The mainline merely accumulates the
Aspects and Features, and starts the app by invoking
{{book.api.launchApp}}:

**`src/app.js`**
```js
import ReactDOM          from 'react-dom';
import {launchApp}       from 'feature-u';
import {routeAspect}     from 'feature-router';
import {reducerAspect}   from 'feature-redux';
import {logicAspect}     from 'feature-redux-logic';
import features          from './feature';

// launch our app, exposing the App object (facilitating cross-feature communication)
export default launchApp({         // *4*

  aspects: [                       // *1*
    routeAspect,   // Feature Routes: Feature.route
    reducerAspect, // redux:          Feature.reducer
    logicAspect,   // redux-logic:    Feature.logic
  ],

  features,                        // *2*

  registerRootAppElm(rootAppElm) { // *3*
    ReactDOM.render(rootAppElm,
                    getElementById('myAppRoot'));
  }
});
```

Here are some **important points of interest** _(match the numbers to
`*n*` in the code above)_:

1. the supplied Aspects _(pulled from separate npm packages)_ reflect
   the frameworks of our run-time stack _(in our example [
   {{book.ext.redux}}, {{book.ext.reduxLogic}}, and
   {{book.ext.featureRouter}})_ and extend the acceptable Feature
   properties _(`Feature.reducer`, `Feature.logic`, and
   `Feature.route` respectively)_ ... _**see:**
   {{book.guide.detail_extendableAspects}}_

2. all of our app features are supplied (accumulated from the
   `features/` directory)

3. a {{book.api.registerRootAppElmCB}} callback is used to catalog the
   supplied `rootAppElm` to the specific React platform in use.
   Because this registration is accomplished by your app-specific
   code, **feature-u** can operate in any of the React platforms, such
   as: React Web, React Native, Expo, etc. ... _**see:**
   {{book.guide.detail_reactRegistration}}_

4. _as a bit of a preview_, the return value of {{book.api.launchApp}}
   is an {{book.api.App}} object, which promotes the accumulated
   Public API of all features.  The App object contains named feature
   nodes, and is exported to provide {{book.guide.crossCom}} ... _here
   is what app looks like (for this example):_

   ```js
   app: {
     featureA: {
       api: {
         open(),
         close(),
       },
     },
     featureB: {
       ...
     },
   }
   ```

Hopefully this gives you a basic feel of how **feature-u** operates.


## Comprehensive Documentation

The sample above just scratches the service!

**Comprehensive Documentation** can be found at https://feature-u.js.org/,
which includes both a **Dev Guide** *(building concepts with full and
thorough **examples**)*, and a complete **API Reference**.


## Benefits

The benefits of using **feature-u** include:

1. **Feature Encapsulation:**
   _isolating feature boundaries improves code manageability_

1. **Feature Collaboration:**
   _allow **Cross Feature Communication** through a well-defined
   feature-based Public Interface_

1. **Application Life Cycle Hooks:**
   _features can initialize themselves without relying on an external
   process_

1. **Feature Enablement:**
    _enable/disable features through a run-time switch_

1. **Minimize Feature Order Dependency Issues**
   _during in-line code expansion_

1. **Framework Integration:**
   _automatically configure used framework(s) (matching the app's
   run-time-stack) by accumulating all feature aspects (employing an
   extendable API)_

1. **UI Component Promotion:**
   _features can autonomously promote their UI components through
   Feature Based Route Management_

1. **Single Source of Truth:**
   _is facilitated in a number of ways within a feature's
   implementation_

1. **Simplified App Startup:**
   _launching an app can be accomplished through a single line of
   executable code!_

1. **Operates in any React Platform**
   _React Web, React Native, Expo, etc._

1. **Plug-and-Play:**
   _features can be more easily added or removed_


## Real Example

Want to see a real **feature-u** app?

{{book.ext.eateryNod}} is the application _where **feature-u** was
conceived_.  It is a {{book.ext.reactNative}} {{book.ext.expo}} mobile
app, and is one of my sandbox applications that I use to test
frameworks.  _I like to develop apps that I can use, but have enough
real-world requirements to make it interesting._

**{{book.ext.eateryNod}}** randomly selects a "date night" restaurant
from a pool of favorites.  _My wife and I have a steady "date night",
and we are always indecisive on which of our favorite restaurants to
frequent :-)_ So **{{book.ext.eateryNod}}** provides the spinning
wheel!






I hope you enjoy **feature-u**, and comments are always welcome.

&lt;/Kevin&gt;



<!--- TODO: ?? retrofit these links ---> 

[feature-u]:               https://feature-u.js.org/
[Getting Started]:        https://feature-u.js.org/start.html
[Basics]:                 https://feature-u.js.org/basics.html
[A Closer Look]:          https://feature-u.js.org/formalTypes.html
[ActionStruct Shapes]:    https://feature-u.js.org/shapes.html
[Parameter Validation]:   https://feature-u.js.org/validation.html
[Defaulting Parameters]:  https://feature-u.js.org/default.html
[Thunk Action Creators]:  https://feature-u.js.org/thunks.html
[Action Promotion]:       https://feature-u.js.org/promotion.html
[Action Documentation]:   https://feature-u.js.org/actionDoc.html
[Distribution]:           https://feature-u.js.org/dist.html
[Why feature-u?]:          https://feature-u.js.org/why.html
[Revision History]:       https://feature-u.js.org/history.html
[MIT License]:            https://feature-u.js.org/LICENSE.html
[API Reference]:          https://feature-u.js.org/api.html
[generateActions]:        https://feature-u.js.org/api.html#generateActions
[ActionNodes]:            https://feature-u.js.org/api.html#ActionNodes
[ActionGenesis]:          https://feature-u.js.org/api.html#ActionGenesis
[actionMeta]:             https://feature-u.js.org/api.html#ActionMeta
[actionMeta.traits]:      https://feature-u.js.org/api.html#ActionMeta
[ActionStruct]:           https://feature-u.js.org/api.html#ActionStruct
[redux]:                  http://redux.js.org/
[actions]:                http://redux.js.org/docs/basics/Actions.html
[action creators]:        http://redux.js.org/docs/basics/Actions.html#action-creators