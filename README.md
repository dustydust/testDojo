# Style guide

This guide covers the coding conventions and practices for HTML, CSS, Javascript, and patterns. It is an evolving and incomplete document. Please edit it to clarify any items and add code examples.

## React Components

- **Component definition**:

  ```javascript
    export default class MyComponent extends React.Component {
      // First class-level statics
      propTypes = {}
      defaultProps = {}

      // Then lifecycle methods. Try to keep these in logical order
      constructor () {}
      componentDidMount () {}
      componentWillReceiveProps () {}

      // The main render function
      render () {}

      // Any partial render functions. Try to keep these in the vertical order that these elements appear on the page.
      _renderThingAtTop () {}
      _renderThingInMiddle () {}
      _renderThingAtBottom () {}

      // Event handlers.  Note the arrow `=>`, which ensures the `this` context.
      _doThing = () => {}

      // Anything else
      _someDataCalculatorThing () {}
    }
  ```

- **Component subrenders**: should be component private methods and prefixed with "render" -  `_renderHeader()`, `_renderModalBody()`, etc.  You may use either if/else or early returns when defining conditional renders.  `undefined` should be returned if nothing is to be rendered.

  ```javascript
  export default class MyComponent extends React.Component {

    render () {
      return (
        <div>
          {this._renderSectionOne()}
          <ul>
            {this.props.items.map(this._renderItem, this)}
          </ul>
          {this._renderSectionTwo()}
        </div>
      );
    }

    _renderItem (item) {
      return <li>{item.text}</li>
    }

    // This example uses ifs.  Note that returning undefined renders nothing.
    _renderSectionOne () {
      if (this.props.foo) {
        return <div>Foo</div>
      } else if (this.props.bar) {
        return <div>Bar</div>
      }
    }

    // This example uses early return.
    _renderSectionTwo () {
      if (!this.props.baz) return;
      return <div>Baz</div>
    }

  }
  ```

- **Component private methods**: prefix with `_` unless explicitly designed to be called by a parent, which is rare (usually limited to things like `focus` on components).

- **Action handler names**: use `on` (`onClick`, `onConfirm`), etc when passing in "handlers", where the component is very generic and does not know what the action will do.  The verb that follows should be a UI action.  Do not use "on" (`createStudent`, `addClass`) when you are a higher-level component that "knows" what the action will do.

- **PropTypes**: These are highly encouraged on all components.  If the same shape is repeated in many components, consider abstracting it (in this case, `post`).  If a component has many PropTypes, consider spacing them logically.
  ```javascript
  // You can import PropTypes to avoid having to type `React.` every time.
  import React, {PropTypes} from "react";

  export default MyComponent extends React.Component
    propTypes = {
      // Try to group properties logically.
      teacher: PropTypes.object.isRequired,
      classroom: PropTypes.object.isRequired,
      students: PropTypes.array.isRequired,

      posts: PropTypes.arrayOf(PropTypes.shape(POST_SHAPE)),
      likes: PropTypes.object.isRequired,
      reads: PropTypes.object.isRequired,
      comments: PropTypes.object.isRequired,

      // Functions should be near the end.
      createPost: PropTypes.func.isRequired,
      likePost: PropTypes.func.isRequired,
      unlikePost: PropTypes.func.isRequired,
      readPost: PropTypes.func.isRequired,
      editPost: PropTypes.func.isRequired,

      // Feature flag stuff comes last, since it will be removed.
      // See "semantic duplication" below...
      hideNewUserBanner: PropTypes.func.isRequired,
    }
  }
  ```

- **Pure Render Functions**: React components have an optional method `shouldComponentUpdate`.  Whenever possible, use `pureRender` for this function.  This will prevent the component from rendering if its parent renders but none of the component's props have changed.  When passing props to components, try to keep this in mind.

  ```javascript
  render () {
    return (
      <Component
        // Defining a new object each time breaks pureRender.
        // Define the object as a constant somewhere else.
        style={{width: "100%"}} // bad
        style={styles.fullWidth} // good

        // Binding functions or creating lambdas breaks pureRender.
        // Use the `autobind` decorator on methods, or bind them in
        // a component's constructor instead.  Note that for components
        // using the old `React.createClass` syntax for definition,
        // a component's methods are bound to it automatically.
        fn={this._method.bind(this)} // bad
        fn={(arg) => this._method(arg)} // bad
        fn={this._boundMethod} // good

        // In general, try to flatten props as much as possible.
        // Passing primitives is easier to debug than objects whose
        // shapes are not guaranteed.  It also makes it harder
        // for changes upstream to break your component.
        teacher={teacher} // bad, unless for very high level components
        teacherId={teacher._id} // good
        teacherName={fullName(teacher)} // good.  this method is in the `name` util!
      />
    );
  }
  ```

- **Feature Flags**: (For teach) Sometimes, small copy changes in views (and not view controllers) depends on a feature flag. You can require the `UserConfigMetadata` store directly to get the feature flag value rather than creating a view controller or passing the feature flag value all the way down. You *don't* need to watch the `UserConfigMetadata` store on that view; feature flags remain constant (except when updated manually with `setFeatureFlag`) and if a user logs out and another user logs back in, the components that depend on those feature flags will have been destroyed and recreated.

- **Semantic Duplication in props**: Try to avoid passing in multiple props that rely on each other.  See the following example.
  ```javascript
  <Component
    // This uses two props where one would do.
    shouldDisplayBanner={this.props.hasMetadataField}
    closeBanner={this._closeBanner}

    // This is preferred.  We can use the presence of the `closeBanner`
    // function to determine whether or not to display the banner.
    closeBanner={this.props.hasMetadataField ? this._closeBanner : null}

  />
  ```

## JSX Style

- **Elements with many props**: indent props 1 line, put closing `>` OR `/>` on new line, unindented

  ```javascript
    <div
      prop1="val1"
      prop2="val2"
      prop3="val3"
    >
      <ComponentWithManyProps
        prop1="val1"
        prop2="val2"
        prop3="val3"
      />
      <ComponentWithFewProps a="b" />
    </div>
  ```

- **Multiline JSX**: Should be wrapped in paretheses and indented.

  ```javascript
  // this is ok
  return <div thing="whatever">{item.text}</div>;

  // but longer things should be indented.
  return (
    <li
      key={index}
      className="list-item"
      onMouseEnter={this._onHover}
    >
      <span>{item.text}</span>
    </li>
  );
  ```

- **JSX ternaries and guards (&&)**: only if each condition fits on one line.

  ```javascript
  // this is ok
  <div>
    {condition && <span>Foo</span>}
    <condition ? <span>Foo</span> : <span>Bar</span>}
  </div>

  // this is bad.  use a conditional render function instead.
  <div>
    {
      condition &&
      <ul>
        <li>One</li>
        <li>Two</li>
        <li>Three</li>
      </ul>
    }
  </div>
  ```

## Javascript

- **If statments**: brackets are not required if the condition and consequent are on the same line.

  ```js
  // this is good
  if (err) return cb(err);

  // this is also good
  if (err) { return cb(err); }

  // this is bad
  if (err)
    return cb(err);
  ```

- **Functions**: prefer function declarations to function expressions, except in anonyous callbacks/iterators, where arrow functions are preferred for brevity.

  ```javascript
// this is a function declaration.  use these.
function myFunc () { /* do stuff */ }

// this is a function expression.  don't use these unless needed.
const myFunc = function () { /* do stuff */ }

// this is an arrow function with implicit return, as an iterator
students.map(s => s.firstName + " " + s.lastName);
```

- **Methods**: Should be defined using ES6 syntax.
  ```javascript
    const myObject = {
      method1 () {

      },
      method2 () {

      }
    }
  ```

- **`var` vs `let` vs `const`**:
  * `const` should be used wherever possible.
  * `let` should be used when needed.  See below for an example
  * `var` should never be used.

  ```javascript
  let text;
  if (condition) {
    text = "hey, who do you think you are, buddy?";
  } else {
    text = "hey, are you having a nice day?";
  }
  ```

- **Module Imports**: Use ES6 style, e.g. `import React from "react";`

- **Promise indentation**:

  ```javascript
  initialPromise()
    .then(function() {
      //stuff
    })
    .then(function() {
      //stuff
    });
  ```

## Filesystem and Naming Conventions

- **Filenames**: same case as you would name its import. That is to say, lowerCamelCase for most things and UpperCamelCase for classes and components.

## BEGIN RAFA'S BRAIN DUMP

### Shared components

We have a top lavel shared folder that contains all our components that can be reused. Think of our shared folder us our toolset our `lodash` for building UI. Having a good set of tools for building our UI speeds up our proces a LOT, BUT there are some things to care about. Adding something to the share folder means that whoever is working on the project can go and start using that thing, before you realised many places in the app might be using a component you put in that folder, therefore lets add things to that folder carefully and with diligence since this is our shared toolset probably the most important part of a UI, lets treat it as such. So a few considerations.

- **Do not rush into making a component generic**, if you have no clear idea of how much is something you are building gonna be reused or you don't have another example yet, well just make it an isolated component but living inside the use case you are developing, lets prevent over engineering here.
- **Be diligent with the stuff you put in the shared folder**, if you are making a component that is intended to be reused well put some comments on whats the intention and what are the responsibilities of it, this will help a lot new people bumping into the component, knowing if that component suits they need or if is valid to refactor them or whatever. If you are building something is in a very early stage, well put a massive comment indicating why you built it whats the idea for it in the future, still very useful for decision making over when to refactor or redo or delete that component.
- **When adding a component to the shared folder PLEASE add it to the live style guide**, again this is our toolset lets make sure we know what tools we have, make sure we don't have duplicated, make sure they work.
- **Search for other components before making a new one**. A lot of times we rush into building a component and we might already have something similar that we can extend or refactor or maybe taht we need to change a bit to make place to our new component and avoid clashing.
- **Remove components that make no sense anymore**, lets walk the extra mile of checking if we can remove a component after we refactor something or we add something that invalidtes an old component, lets make sure we remove lines of code in our toolset while keeping the same functionality and not add line of codes for equal functionality.

The toolset of a UI does not change much after the core is built, is like lodash methods once the main methods are there then you just tune them, bug fix them, etc. Well this is the same, lets make sure we grow our components organically with a similar api with a similar coding style, paying attention to the points above is very important for delivering a consistent set of shared components.

### Palette and standard layout units.

#### Palette.
Our goal is to never ever write colors manually by its hash or rgb or whatever, we should ALWAYS use the shared palette living in common_styles. Why?, well the idea is to have a standardize color palette, automating this might not be worth or might not be easy but there is something very important to follow here! which is keeping in sync with design team, every single color in our shared common palette should be known and in sync with the palette the design team has. This means that if you see a new color somewhere in a mock up PLEASE do not go and add it to the palette without asking, ask the design team if that new color should be part of our guide or if it simple a mistake or even if it's just used in one single place, if so then just write down manually the color in that place, but DO NOT modify the shared palette without checking with design.

#### Standard layout and units.
We built everything on top of Layout components, which means that if you have to position something in the screen there will be a component for you, Layout, Container or Grid probably will do it (check the live styleguide for examples). There are quite a few benefits of doing this but an important one is using standard gutter and grid size, all our layout components take a multiplier and not a fixed size, this mean we should NOT write margins or paddings manually with custom sizes when doing layout, this way we can rely on an internal size that we can modify if we need and more importantly that is consistent across all the app. SO PLEASE do not do margins and paddings on layout manually, if by any chance a layout component does not suit you you could use the gridSize and gutterSize from our common styles but should not be that common.

#### Gutter vs padding

Gutter is whitespace between sibling elements.

Padding is whitespace within a container, around child elements.

It's an anti-pattern to set gutterTop/Left on the first LayoutItem or gutterBottom/Right on the last LayoutItem. Use the container's (i.e. HLayout or VLayout) padding instead because the whitespace in question is between the container and the child elements, not between sibling elements.

### Inline styles

We are using inline styles in all places that is possible.
We are using [react-style](https://github.com/js-next/react-style) for providing support to it (like extracting classes when deploying to production and media queries support).
So a few conventions when writing styles in your components or pages:

* If possible write them inline DO NOT USE stylus sheets unless really necessary.
* When writing styles use react-style for the static styles and normal literals for the dynamic styles. This is very IMPORTANT! otherwise react style will be regenerating the styles every time we render which is the same as not using it. We want to use the same naming convention for doing it, check it out here.

  ```js
  let Component = React.createClass({
    render() {
      let dynamicStyles = getDynamicStyles(this.props);
      return <div styles={[styles, dynamicStyles]}></div>
    }
  })

  // Totally fine to use variables
  let boxSize = '10';
  let styles = ReactStyle.create({
    width: boxSize + 'rem',
    padding: boxSize / 2 + 'rem',
    color: white
  });

  function getDynamicStyles(props) {
    return {height: props.height}
  }
  ```

  Whatever is static we would put it in a styles variable and then use whatever we want from inside it,      like styles.container style.listItem, etc.
Whatever might change based on user input or any later computation we will put it inside a method that will return that style just as a literal and that will merge on the styles of the component.

  Another classic case is static styles but picking one or the other at render time. For this you can use the following pattern and is fine since react style is gonna replace `styles.boxHovered` with the class name generated at transpile time.
  ```js
  let Component = React.createClass({
    render() {
      return <div styles={[styles.box, this.state.hovered && styles.boxHovered]}></div>
    }
  })

  let styles = ReactStyle.create({
    box: {
      width: '10rem'
    },
    boxHovered: {
      border: '1px solid red'
    }
  });
  ```


# Using a Vagrant VM to run API locally

If you're running Ubuntu as your host OS, there are more specific instructions [here](https://github.com/classdojo/tech-wiki/wiki/API-Installation---Ubuntu---Vagrant)

## Install VirtualBox
https://www.virtualbox.org/wiki/Downloads

Make sure to grab a version >= 5.1.14

Note: To get frontend tests working on Ubuntu, I had to add a port-forwarding entry for the relevant ports in Oracle VM VirtualBox Manager -> api box -> network -> port-forwarding.

## Install Vagrant

Versions >= 1.9.1 should work. https://releases.hashicorp.com/vagrant/

If you see issues with vagrant starting up, try this:
```
vagrant reload
```

## Install ansible

Version >= 2.1 should work.

http://docs.ansible.com/ansible/intro_installation.html

## Setup VM

`make up-dev`

This will take a while. Vagrant will download the base VM and then run some provisioning on top to make sure the development environment is set up.

**NOTE:** If you run into the following error on Mac OSX

`Unexpected Exception: (setuptools 1.1.6 (/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python), Requirement.parse('setuptools>=11.3'))`

Then run the following command:

`pip install --upgrade setuptools --user python`

**NOTE:** In the future, whenever someone makes changes to the provisioning scripts, simply pull the latest API code and do `make provision-dev`.

## SSH into the VM and start up the API

```
vagrant ssh
cd /vagrant
rm -rf node_modules # only when using an existing API repo
yarn install
NODE_ENV=dev make fixtures
NODE_ENV=testing make fixtures
make run
```

The `/vagrant` directory is synced with the `api` directory on your host machine.
