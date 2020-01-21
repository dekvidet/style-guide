**This guide explains how files and directories should be structured and named.**

  > The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Table of Contents

  1. [Component and File Naming](#component-and-file-naming)
  1. [App Structure](#app-structure)
  1. [Scenes and Components](#Scenes-and-components)
  1. [Reducers](#reducers)
  1. [Sagas](#sagas)
  1. [API](#api)
  1. [Services](#services)
  1. [Constants](#constants)
  1. [Assets](#assets)

## Component and File Naming

  - **File and directory names:** MUST use lowerCamelCase except for React components, containers, sagas, ducks and their directories which should use UpperCamelCase (PascalCase). 
  - **Component and scene names** MUST consist of the following forumla: `noun + component type`.
    - Noun: A component is a thing that displays something and/or lets you execute an action. Its "thing" aspect should be described. If you have problems finding a noun just use the magic `er` suffix or reword to a more specific meaning.
    - Component type: Your component should belong to one of the types listed below, and its name SHOULD be suffixed with its type except when it is of "view" or "scene" type. Everything without a type suffix is assumed to be of type "view" or "scene" so that suffix MUST NOT be added.
      - View: If a component's only role is to displays something. If a component initiates API calls, it MUST NOT be a view type.
      - Form: A component functioning mainly as a form.
      - Button: A component starting an action on click.
      - List: A component that renders an iterable.
      - Field: Any kind of form field.
      - Wrapper: [HOC](https://reactjs.org/docs/higher-order-components.html)s. The "noun" part of the name SHOULD indicate what type of components it wraps.
      - Scene: *Only* for scene components that map the URI hierarchy.
      - Container: MUST be used and *only* by component wrappers that use `react-redux`'s `connect()` to inject `mapStateToProps` and/or `mapDispatchToProps` to a component. The "noun" part of the name MUST be the same as the full name of the wrapped component. So `ItemListContainer` for `ItemList`, `WelcomeSceneContainer` for `WelcomeScene`.
  - Every component's CSS style MUST be stored in a `*.css` file in the same directly with the same name as the component. So `ItemList.css` for `ItemList.js`.
  - Every component's sagas MUST be stored in a `*Saga.js` file in the same directly with the same name as the component. So `ItemListSaga.js` for `ItemList.js`.
  - Every reducer's, service's and constant's filename MUST be a noun.
  - It is RECOMMENDED to not have two components with the same name. This removes obscurity and improves IDE [fuzzy search](https://en.wikipedia.org/wiki/Approximate_string_matching)(`Ctrl+P` in most IDEs). If your component's desired name is already taken then that signals a [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself) violation or unspecific naming.
  - Every scene's and component's directory MUST have an `index.js` file that exports the directory's content.
  - You MUST export a component with the same name in an `index.js` except when you export a container component of a component in which case the `Container` suffix MUST be omitted.
    ```jsx
    // bad
    export { default as LoginDialog } from './LoginContainer'
    export { default as ItemContainer } from './ItemContainer'

    // good
    export { default as LoginDialog } from './LoginDialog'
    export { default as Item } from './ItemContainer'
    ```

## App Structure

  This is an overview of the directory hierarchy.

  ```
  repo-root
  ├── app: root folder for application code
  │   ├── scenes: route paths mapped to a specific component
  │   ├── components: application-wide reusable components
  │   ├── reducers: application-wide reducers that doesn't belong to any scene 
  │   ├── sagas: application-wide sagas that doesn't belong to any scene
  │   ├── services: Utility, helper functions and libraries
  │   ├── constants: JS and JSON files holding data
  │   └── assets: multimedia files like images, videos, sounds
  ```

## Scenes and Components

  For **general scenes**, the scene's path should map the route's URI, so the `admin/dashboard` route's scene should be in `app/scenes/admin/Dashboard/Dashboard.js`. The URI and the directory path should be the same except for the last directory where the name should be UpperCamelCase because that is the component's name. [URIs should be written in kebab-case](https://support.google.com/webmasters/answer/76329).

  When a sceen works on a **CRUD resource** you SHOULD follow the pattern below:

  | URI                      | Action  | Component path & name         |
  | ------------------------ |-------- |------------------------------ |
  | admin/posts              | index   | scene/admin/posts/PostIndex   |
  | admin/posts/create       | create  | scene/admin/posts/PostCreate  |
  | admin/posts/:id          | show    | scene/admin/posts/PostShow    |
  | admin/posts/:id/edit     | edit    | scene/admin/posts/PostEdit    |
  | admin/posts/id-generator | custom* | scene/admin/posts/IdGenerator |

  - A scene is a component whose directory hierarchy is the same as the URI path that instantiates it.
  - Only a scene can directly access the Redux store. Components can only get Redux store data trought their props.
  - Everything that isn't a scene is a component. This usually means reusable components, although a component can be a singleton as well.
  - A scene or component can have subcomponents which MUST reside in a subdirectory called `components`.
  - A scene can have subscenes which MUST reside in a subdirectory called `scenes`.
  - Every scene or component can use the components inside their directories but not the other way around.
  - Every scene can use the subscenes inside their directories but not the other way around.
  - Every scene or component can have its own `services.js` or `constants.js` but only if the functionality in them is useful for other components or subcomponents in that directory and if it isn't useful for other components outside of the directory because those should be stored in the `app/constants` and `app/services` directories.
  - A components `services.js` and `constant.js` can be imported by all of its subcomponents and their `services.js`. 
  - Application-wide components in `app/component` can be used anywhere. Every component in that directory SHOULD reside in their own directory (even if it will only have a single file in it). Similar application-wide components MAY be grouped into directories in which case an additional level can be introduced e.g. `app/components/dialogs/ConfirmationDialog`.
  - A scene's duck can be used everywhere inside its subdirectories so subscenes can dispatch its actions.
  - A component inside the `app/components` directory MUST NOT have any reducers, ducks or sagas. Application-wide sagas and reducers are stored in the `app/sagas` and `app/reducers` directories.

### Example

  The following directory structure is an example of a solution that adheres to this guide. Further down you can find explanations for this specific case.

  ```
  app
  ├── scenes: components structured according to URIs
  │   ├── Dashboard
  │   │   ├── components
  │   │   │   ├── DashboardTitle
  │   │   │   │  ├── DashboardTitle.js
  │   │   │   │  └── index.js
  │   │   │   └── DashboardAddButton
  │   │   │      ├── DashboardAddButton.js
  │   │   │      └── index.js
  │   │   ├── Dashboard.js
  │   │   ├── DashboardContainer.js
  │   │   ├── DashboardSaga.js
  │   │   ├── DashboardDuck.js
  │   │   ├── Dashboard.css
  │   │   └── index.js
  │   ├── admin
  │   │   └── posts
  │   │       ├── PostIndex
  │   │       │   ├── PostIndex.js
  │   │       │   ├── PostIndexContainer.js
  │   │       │   ├── PostIndexSaga.js
  │   │       │   ├── PostIndexDuck.js
  │   │       │   ├── PostIndexContainer.css
  │   │       │   └── index.js
  │   │       └── PostEdit
  │   │           ├── components
  │   │           │   └── PostSummary
  │   │           │      ├── PostSummary.js
  │   │           │      ├── PostSummaryContainer.js
  │   │           │      └── index.js
  │   │           ├── PostEdit.js
  │   │           ├── PostEditContainer.js
  │   │           ├── PostEditSaga.js
  │   │           ├── PostEditDuck.js
  │   │           ├── constants.js
  │   │           ├── services.js
  │   │           └── index.js
  │   ├── profile
  │   │   └── Organizations
  │   │       ├── Organizations.js
  │   │       └── index.js
  │   ├── Main: root route
  │   │   ├── Main.js
  │   │   └── index.js
  │   └── AboutUs
  │       ├── AboutUs.js
  │       └── index.js
  ├── components: application-wide reusable components
  │   ├── dialogs:
  │   │   ├── ErrorDialog
  │   │   │   ├── ErrorDialog.js
  │   │   │   └── index.js
  │   │   └── ConfirmationDialog
  │   └── Counter
  │       ├── Counter.js
  │       └── index.js
  ```

   - `ErrorDialog` and `Counter` can be used anywhere in the app even in the `app/components` folder.
   - `DashboardAddButton.js` can't use other components from the app except from `app/components`.
   - `PostEdit`'s `constants.js` and `services.js` can be used by `PostSummary.js` or other files in that directory. 
   - `PostSummaryContainer.js` can import the action types from `PostEditDuck`.

## Reducers

  The `reducers` directory holds the files that combine all the scenes' reducers and every reducer that doesn't belong to any specific scenes or are being used by multiple ones.

## Sagas

  The `sagas` directory holds the files that combine all the sagas' reducers and every reducer that doesn't belong to any specific scenes or are being used by multiple ones.

## API

  The `api` directory holds all the files necessary to communicate with the backend's API.

  ```
  app
  ├── api
  │   ├── routes.js: list of available backend API endpoints/routes
  │   └── services.js: functions to help fetch data
  ```

## Services

  The `services` directory holds functions that only contain logic and not directly related to displaying UI. functionality should be grouped to general files and exported from there. You MAY create directory to group large amount of funcitonality that a single file can't comprehend e.g. bigger libraries. In this case the directory should be named by that feature/library and every file should be conteined directly in them.
  You MUST NOT use `index.js` files in the `services` directory or its subdirectories. Everything MUST be imported directly from its file. Services MUST NOT `export default`. A service file is analogous for a directory of component, but it is a collection of functions so there is no need for `index.js` or for defaults.

  ```
  app
  ├── services
  │   ├── formatting.js
  │   ├── dateTime.js
  │   └── searching
  │       ├── textSearchLenses.js
  │       └── imageSearchLenses.js
  ```

## Constants

  The `constants` directory holds static data source files that are used/executed as some type of source code. This usally means JS or JSON files that hold 

  JavaScript files MUST NOT contain or export anything other than strings, objects or arrays.

  ## Assets

  The `assets` directory holds every static resource whose content is not used/executed as source code. They can be processed by source code but they are inputs not dependencies of them. Usually this means that you put multimedia files here.

  It is NOT RECOMMENDED to store assets anywhere in the `app` directory aside from the `assets` directory.
