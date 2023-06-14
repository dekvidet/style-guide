# JS-React-Redux Style Guide (Draft)

  > The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Table of Contents

  1. [Prerequirements](#prerequirements)
  1. [Naming](#naming)
  1. [General Style](#general-style)
  1. [Container Style](#container-style)
  1. [Component Style](#component-style)
  1. [Duck Style](#duck-style)
  1. [Ordering](#ordering)

## Prerequirements

  - You MUST follow the [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
  - You MUST follow the [Airbnb React/JSX Style Guide](https://github.com/airbnb/javascript/tree/master/react)
  - You MUST follow the [Ducks: Redux Reducer Bundles](https://github.com/erikras/ducks-modular-redux) pattern
  - You MUST follow the [Flux Standard Action](https://github.com/redux-utilities/flux-standard-action)
  - You MUST follow our [Directory Structuring Style Guide](directory-structuring-guide.md)

If anything in this style guide conflicts with the 3rd party guides, then this guide SHALL overwrite the 3rd party one(s).

## Naming

Names should be unique, identifiable and clearly communicate the intent. A good name replaces a comment describing what the actual thing does. 

  - **Generic naming:** You MUST NOT use:
   - single letter names (`a`, `b`, `c`) because they are hard to understand and impossible to search for (some exceptions are discussed later)
   - names followed by incrementing numbers (`car1`, `car2`, `car3`)
   - confusingly similiar names (`computationalNodeProcessingDataCreator`, `computationalNodeProcessingDataEditor`) because they are easy to mix up
   - names that try to imitate a reserved language keyword (`dis`, `klass`).
  - **Part of speech**: Variable names, classes, components SHOULD be nouns, while methods, functions, procedures SHOULD be verbs.
  - **Measurement variables:** If a variable holds a measurement, it's name SHOULD containt the masurement's subject and units like `litresOfConsumedWater`, `lenghtOfFieldInMeters`. For integer measurement of collections and strings use the `Count` and `Length` suffixes respectively, for example:  `eventCount`, `nameLength`.
  - **Numeric constants:** When you need to use a numeric constant (unless it is trivial like divison by 2 for parity or comparing to 0 when something is empty) you MUST create a SCREAMING_SNAKE_CASE `const` variable for it and use that.
  - **Functions returning booleans**: When a function returns a boolean its name should start with a verb indicating this nature: `hasChildren`, `isEven`, `canUpdate`.
  - **Casing:** In case of camelCase and PascalCase naming every 2nd or so word's letters MUST be lovercase except the first letter of the word. This applies to abbervations and acronyms too. The only reason to have 2 or more uppercase letter next to eachother is when one of the letters is a single letter word like "I" although it is RECOMMENDED to not use single letter words in names.
    ```jsx
    // bad
    fetchAPI()
    ISOToUTF()

    // good
    fetchApi()
    IsoToUtf()

    // acceptable
    generateULockId()
    ```
  - **Numbers:** It is RECOMMENDED to avoid numbers in names.
    ```jsx
    // bad
    string2Int()

    // good
    stringToInt()
    ```
  - **Parameters:** When working with [parameters and arguments](https://stackoverflow.com/a/1788926) you MUST name your variables accordingly. You pass arguments to a function and a function receives parameters.
    ```jsx
    // bad
    const calculate = (args) => {
      switch (args.length) { ... }
    }
    const renderProfile = () => {
      const profileParams = pick(this.state.data, ['foo', 'bar'])
      return <Profile {...profileParams} />
    }

    // good
    const calculate = (parameters) => {
      switch (parameters.length) { ... }
    }
    const renderProfile = () => {
      const profileArguments = pick(this.state.data, ['foo', 'bar'])
      return <Profile {...profileArguments} />
    }
    ```
  - **Subrenderers:** If a method is used for rendering/returning HTML then its name MUST be prefixed with `render` e.g., `renderAvatar()` instead of just `avatar()`.
  - **Names of component props:** Event handlers MUST be prefixed with `on` e.g., `onClick` instead of `click`, `clickHandler`, `handleClick`.
  - **Event handler names:** MUST be constructed like this: `handle` + name of the subject of interaction + event type. If the handler is used for multiple events the event type SHOULD be replaced with `Events`: 
    ```jsx
    // bad
    onClick()
    handleChange()
    handleCanvasEvents() // if handles only 1 event

    // good
    handleAddButtonClick()
    handleDescriptionChange()
    handleCanvasEvents() // if handles at least 2 event
    ```
  - The parameters of `mapStateToProps` and `mapDispatchToProps` MUST be named exactly like this:
    ```jsx
    // good
    const mapStateToProps = (state, ownProps) => {}
    const mapDispatchToProps = (dispatch, ownProps) => {}
    ```
  - **Single letter variable names:** You SHOULD avoid naming variables to single letters except for really generic things like `i` in for loops, `a` and `b` in comparing arrow functions, or the first letter of an entity in a single line arrow function. For nested loops you SHOULD NOT use single letter variable names unless you traverse a multi dimensional entity in which case `x`, `y`, `z` (maybe `i`, `j`, `k`) are acceptable.
    ```jsx
    // bad 
    interviews.filter(i => {
      if (i.managed_by === this.props.currentUser.name) {
        checkAvailability
      }
      return false
    })

    // ok
    cars.filter(c => c.managed_by === this.props.currentUser.name)

    // encouraged
    interviews.filter(interview => interview.managed_by === this.props.currentUser.name)

    // ok
    for (let i = 0; i < items.length; ++i) {
      items[i] = items[i].sort((a, b) => a.count > b.count)
    }
    ```

## General Style

  - **Arrow functions:** are RECOMMENDED if the function doesn't require its own `this` and can be written using the immediate return shorthand (`() => 5` instead of `() => { return 6 }`) for arrow functions. In other cases the easier to read, then the shorter approach is prefered.
    ```jsx
    const renderAvatar = src => <div><img src={src} /></div>
    ```
  - **Early returning:** is RECOMMENDED instead of caching the result and returning at the end of the function. However if it doesn't harm code readability try to express the functionality in a single `return` in a functional manner.
    ```jsx
    // discouraged
    const calculatePrice = data => {
      const now = new Date()
      let result = null
      if (data.dueDate > now) {
        result = data.totalWithInterest(now) + PENALTY_AMOUNT
      }
      result = data.total()
      return result 
    }

    // ok
    const calculatePrice = data => {
      const now = new Date()
      if (data.dueDate > now) {
        return data.totalWithInterest(now) + PENALTY_AMOUNT
      }
      return data.total()
    }

    // best
    const calculatePrice = data => {
      const now = new Date()
      return data.dueDate > now ? data.totalWithInterest(now) + PENALTY_AMOUNT : data.total()
    }
    ```
  - **Destructuring `state` and `props`:** You SHOULD destructure `this.state` and `this.props` if the destructuring part uses less characters than the parts being removed thanks to its introduction. This usually happens if a variable/function is used at least 2 or 3 times.
    ```jsx
    // discouraged
    handleItemSearchClick = () => {
      const { items } = this.props
      const { isFiltered } = this.state

      const sortedItems = items.sort((a, b) => a.count > b.count)
      if (isFiltered) {
        return sortedItems.filter(sortedItem => sortedItem.status == 'active')
      }
      return sortedItems
    }

    // encouraged
    handleItemSearchClick = () => {
      const sortedItems = this.props.items.sort((a, b) => a.count > b.count)
      if (this.state.isFiltered) {
        return sortedItems.filter(sortedItem => sortedItem.status == 'active')
      }
      return sortedItems
    }
    ```
  - **Temporary variables:** You SHOULD create temporary/shortcut variables for code that:
    - Is used more than once - to implement DRY.
    - Needs explaining due to its complexity - to increase readability.
    - Needs specific names in the given context - to increase readability.
  - **Negating in conditions:** You SHOULD avoid negation in conditions that use both the `if` and `else` part. Instead of negation just flip the `if` else `else`:
    ```jsx
    // discouraged
    if (!isImplemented) {
      implement()
    } else {
      use()
    }

    // encouraged
    if (isImplemented) {
      use()
    } else {
      implement()
    }
    ```
  - **Post increments:** You SHOULD NOT use post increments (or decrements). The only case where its usage would be acceptable is if you try to indicate in an expression that the given variable have a side effect of incrementing after evaluation, but since we would like to avoid introducing side effects, in those cases incrementing should happen in a different step where preincerementing should be use.
    ```jsx
    // discouraged
    const result = count++

    // encouraged
    const result = count
    ++count
    ```
  - **Output arguments:** A function SHOULD NOT modify [pass by "copy of a reference"](https://stackoverflow.com/a/13104500/1494454) parameters (objects properties). In JS if a parameter is an object (unless you redefine the parameter) if you modify it (its properties) this will change the object that was passed as an argument to the function. While this can be used as a technique it is usually discouraged since it is a side effect that might become unnoticed. Unless it is required for considerable performance optimization, you SHOULD return a copy of the object or array.
  - **Modifying parameters:** It is RECOMMENDED to not modify a function's parameters. They should be treated as immutable so they can act as a source of truth for that function.

## Duck Style

  - **Action types:** MUST come first and their string value should use this formula: `app/` + path of the directory where the file is stored after `app/scenes/` in [kebab-case](https://stackoverflow.com/a/12273101) + name of the constant. You MAY use your own app's name instead of `app`. For the file `/app/scenes/Dashboard/components/CardStack/CardStackDuck.js`
    ```jsx
    // good
    export const LOAD_ITEMS = 'app/dashboard/components/card-stack/LOAD_ITEMS'
    ```
  - **Action type names:** MUST be verbs describing the effect they make.
    ```jsx
    // bad
    export const STATISTICS = ...
    export const FIELD_CHANGE = ...

    // good
    export const LOAD_STATISTICS = ...
    export const UPDATE_FIELD = ...
    ```
  - **Initial state:** MUST be set in a separate variable then passed to the reducer in this manner:
    ```jsx
    // good
    const initialState = {
      items: false,
    }

    export default function (state = initialState, action) {
      ...
    }
    ```
  - **Main reducer:** The action payload MUST be "copied" to a `const` variable called `p`, except when the payload is not used, in which case it MUST be ommited. A `switch` statement SHOULD be written in the style above.
    ```jsx
    // good
    export default function (state = initialState, action) {
      const p = action.payload

      switch (action.type) {
        case LOAD_ITEMS:
          return update(state, {
            items: { $set: p.items },
          })
        default:
          return state
      }
    }
    ```

## Container Style

  - `mapStateToProps` and `mapDispatchToProps` MUST be an arrow function and should use the immediate return shorthand if they only return an object.
    ```jsx
    // bad
    function mapStateToProps(state) {
      return { currentUser: state.currentUser }
    }
    const mapDispatchToProps = (dispatch) => {
      return {
        setUser: user => dispatch(createAction(SET_USER)({ user })),
      }
    }

    // good
    const mapStateToProps = state => ({ currentUser: state.currentUser })
    const mapDispatchToProps = dispatch => ({
      setUser: user => dispatch(createAction(SET_USER)({ user })),
    })
    ```
  - You MUST ommit all the unused parameters for `mapStateToProps` and `mapDispatchToProps`, except when you have to add the 1st parameter for the sole reason that you can use the 2nd.
  - You MUST NOT pass a second parameter to `connect` if you don't use `mapDispatchToProps`.
    ```jsx
    // bad
    const mapDispatchToProps = (state) => {}
    export default connect(mapStateToProps, mapDispatchToProps)(Profile)

    // good
    export default connect(mapStateToProps)(Profile)
    ```
  - You MUST pass `null` as a first parameter to `connect` if you don't use `mapStateToProps`.
    ```jsx
    // bad
    const mapStateToProps = (dispatch) => {}
    export default connect(mapStateToProps, mapDispatchToProps)(Profile)

    // good
    export default connect(null, mapDispatchToProps)(Profile)
    ```

## Component Style

  - Subsequent multi-line JSX/component expressions on the same indentation level SHOULD be separated by a single empty line if they wrap something, and only between them:
    ```jsx
    // bad
    <div>

      <div>
        <SomeComponent />
      </div>
      <div>
        <SomeComponent />
      </div>

    </div>

    // good
    <div>
      <div>
        <SomeComponent />
      </div>

      <div>
        <SomeComponent />
      </div>
    </div>
    ```
  - Importing a module from the same or a subdirectory MUST use a relative path.
  - Importing a module outside a directory MUST use an absolute path.
    ```jsx
    // bad
    import Counter from '../../components/Counter'

    // good
    import Counter from 'components/Counter'
    ```
  - Components and scenes MUST be imported through their `index.js`
    ```jsx
    // bad
    import Counter from '../../components/Counter'

    // good
    import Counter from 'components/Counter'
    ```
  - **Rendering nothing in a subrenderer:** `undefined` is not rendered by React so just ommit the `return`.
    ```jsx
    // bad
    const renderLabel = (data, isDisplayed) => {
      if (isDisplayed && data.isValid) {
        return <div>{data.items.join(', ')}</div>
      } else {
        return null
      }
    }  

    // good
    const renderLabel = (data, isDisplayed) => {
      if (isDisplayed && data.isValid) {
        return <div>{data.items.join(', ')}</div>
      }
    }  
    ```
  - **Passing render props:** Passing components or other DOM elements via a render prop should happen through a render function.
    ```jsx
    // discouraged
    <Footer
      slogan={
        <div>
          <Logo />
          <p>Cookin' spaghetti code since '98</p>
        </div>
      }
    />

    // good
    <Footer
      slogan={renderSlogan()}
    />

    ...

    renderSlogan = () => (
      <div>
        <Logo />
        <p>Code kitchen with the best servers</p>
      </div>
    )
    ```
  - **Rendering nothing in `render`:** MUST happen via `return null` (React requires a return value in `render` in all cases).
  - **Complex conditions:** SHOULD be avoided for conditional rendering. Instead `{expression && <Component />}` use a separate method.
    ```jsx
    // bad
    {condition1 && (condition2 || condition3) && (
      <Component>
        ...
      </Component>
    )}

    // better
    {this.renderComponent()}

    ...

    renderComponent() {
      if (condition1 && (condition2 || condition3)) {
        return (
          <Component>
            ...
          </Component>
        )
      }
    }
    ```

## Structuring Guidelines 

The topic of what should be a component is highly debated and is a problem that is hard to standardize, so the following section will collect our guidelines and goals/prefered directions in this matter.

- Render method: By default start with putting everything to the `render` method and move functionality when needed. 
- Subrenders: Use subrenders when there is a functional need to move youre code out from the `render()` method:
 - Preventing code duplication.
 - Shorter/more readable.
- Other components: If a subrenderer can be used between multiple components (or has the future potential) then it should be a component. Components should be minimal and complete. Try to avoid small components. When splitting components try to avoid creating dozens of subcomponents and aim to a few.

## Ordering

  - **Ordering for file content**:

  1. Imports (see detailed ordering)
  1. Module/file level constants
  1. Module/file level functions
  1. React component with all of their methods (see detailed ordering).
  1. Component proptypes
  1. Component default props
  1. Exporting the component

  - **Ordering for imports**: The ordering MUST go from the most generic import to the most specific. Starting with 3rd parties and finishing with the most specific import to the importing component. Generally, the deeper an imported feature is in the folder hierarchy the closer it SHOULD be to the bottom of the imports.

  1. React and React related 3rd party imports.
  1. Utility 3rd party imports.
  1. Other 3rd party imports.
  1. UI component imports.
  1. Icon component imports.
  1. App utility/helper imports.
  1. Global app component imports.
  1. Component imports outside of the importing component's folder
  1. Component imports from the importing component's folder (a.k.a., its subcomponents) 
  1. Importing `./styles.css` MUST be the last import (if the component has such a file)

  - **Ordering for `class extends React.Component`:** This section is modifying the [order described in the Airbnb React/JSX Style Guide](https://github.com/airbnb/javascript/blob/master/react/README.md#ordering)):

  1. optional `static` methods
  1. `constructor`
  1. `getChildContext`
  1. `componentWillMount`
  1. `componentDidMount`
  1. `componentWillReceiveProps`
  1. `shouldComponentUpdate`
  1. `componentWillUpdate`
  1. `componentDidUpdate`
  1. `componentWillUnmount`
  1. `render`
  1. *optional render methods* like `renderNavigation()` or `renderProfilePicture()`
  1. *clickHandlers or eventHandlers* like `handleClickSubmit()` or `handleChangeDescription()`
  1. *getter methods for `render`* like `getSelectReason()` or `getFooterContent()`

https://hackernoon.com/react-components-naming-convention-️-b50303551505
https://engineering.musefind.com/our-best-practices-for-writing-react-components-dec3eb5c3fc8
https://medium.com/@alexmngn/how-to-better-organize-your-react-applications-2fd3ea1920f1