# React Redux Intermediate

[![Greenkeeper badge](https://badges.greenkeeper.io/chasm/didactic-doodle.svg)](https://greenkeeper.io/)

This is an intermediate-level tutorial (tied in with a [Codementor.io course](https://www.codementor.io/classes)) that presumes that readers are proficient with JavaScript, have at least a passing familiarity with JavaScript 2015 syntax, and have done some basic work with React.js.

## Instructions

1. Create a git repo for your project and clone it to your working folder.
2. Add or update the `.gitignore` file.

  ```
  .DS_Store
  node_modules/
  *.log
  ```

3. Run `npm init -y` then update the `package.json` file, or leave off the `-y` and answer the questions at the command line.
4. Install webpack locally as a development dependency with `npm i -D webpack` (`i` is short for `install` and `-D` is the same as `--save-dev`).
5. Create the folders and files we'll need:

  ```sh
  mkdir app
  mkdir app/components
  mkdir build
  mkdir test
  touch app/index.jsx
  touch app/main.css
  touch app/components/app.jsx
  touch build/bootstrap.css
  touch build/index.html
  touch test/mocha.opts
  touch test/setup.js
  touch test/tests.js
  touch webpack.config.babel.js
  touch .eslintignore
  touch .eslintrc
  touch .editorconfig
  ```

  There are as many ways to set up your folder hierarchy as there are developers, it seems. This way is pretty arbitrary and I've used many others. Some prefer to call the `app` folder `src`. Some prefer to call the `build` folder `dist` or `public` or (confusingly) `app`. Some use the `jsx` extension, others prefer to keep it to `js`. The `main.css` file could be `styles.css` or `index.css` or `app.css` or something else I haven't thought of.

  Some developers like to use `PascalCase` or `camelCase` for the file names. As not all filesystems are case sensitive, I'm not a huge fan of that practice (though I've used it occasionally). I prefer `train-case`. The important point is to be *consistent*. And if your team has an established style guide, then follow it precisely. Development is a team effort; express your individuality somewhere else.

6. Add the HTML in `/build/index.html`.

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="utf-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1">

      <title>Didactic Doodle</title>

      <link rel="icon" href="/favicon.png">

      <link rel="stylesheet" href="/bootstrap.css">
    </head>
    <body>
      <script src="/app.js"></script>
    </body>
  </html>
  ```

  Some developers like to add a `<div/>` element into which they can render the React DOM. I prefer to create that element on the fly (see `/app/index.jsx` below) and append it to the body before rendering. YMMV.

7. Add the `bootstrap.css` from [Bootswatch](http://bootswatch.com/paper/). At the request of a student, I've switched to the Paper theme. Choose your own, then select the `bootstrap.min.css` link from the navbar and copy and paste the code into your `/build/bootstrap.css` file. You can name this whatever you want. If you're using Sass or LESS, feel free to set this up differently. I'm going to use Myth, so I'm fine with plain CSS, and I'm OK, for now, with just loading it statically.
8. Add some CSS from [myth.io](http://www.myth.io/). We'll use this later to test whether the Webpack `myth-loader` is working (and to demonstrate how to process CSS files in Webpack). Here we're creating a couple of variables, one for the color red and the other to set a "large" pixel value for padding. Then we use them to set the style for paragraphs.

  ```css
  /* app/main.css */
  :root {
    --red: #ff0000;
    --large: 10px;
  }

  p {
    color: var(--red);
    padding: var(--large);
  }
  ```

9. In his book [_Webpack and React_](https://survivejs.com/), Juha Lindstedt creates a small node module to merge configurations in Webpack. We'll use his technique as it is simple and keeps the configuration in one file.

  ```sh
  npm i -D webpack-merge
  ```

10. We're writing our code (most of it) in JavaScript 2015, also known as ECMAScript 6 or ES6 or ES2015. Fun, eh? We'll need to "transpile" this code back to ES5 for it to run in today's browsers. This is the primary reason we're using Webpack. The module we use to do the transpilation is [Babel](https://babeljs.io/). We'll need to install the dependencies we need (as devDependencies).

  ```sh
  npm i -D babel-core babel-loader babel-preset-es2015 \
  babel-preset-react babel-preset-stage-1 babel-register
  ```
  1. [babel-core](https://github.com/babel/babel) provides the core logic of Babel
  2. [babel-loader](https://github.com/babel/babel-loader) is a Webpack loader that runs Babel for us
  3. [babel-preset-es2015](https://babeljs.io/docs/plugins/preset-es2015/) adds the plugins we need to transpile from ES6 to ES5
  4. [babel-preset-react](https://babeljs.io/docs/plugins/preset-react/) adds the plugins we need to work with React
  5. [babel-preset-stage-1](https://babeljs.io/docs/plugins/preset-stage-1/) adds the [Stage 1](https://babeljs.io/docs/plugins/#stage-x-experimental-presets-) plugins (we take a small risk with using features this far out)
  6. [babel-register](https://babeljs.io/docs/usage/require/) adds the require hook, which will bind itself to node's require and automatically compile files on the fly. I mistakenly said this included the polyfill during the class, but it does not. My bad.

11. Configure webpack. I've renamed the config file to `webpack.config.babel.js` to take advantage of the `babel-register` require hook. Now we can use imports and other ES6 syntax in our config file. So yay.

  ```js
  // webpack.config.babel.js
  import merge from 'webpack-merge'
  import path from 'path'

  const TARGET = process.env.npm_lifecycle_event

  process.env.BABEL_ENV = TARGET

  const PATHS = {
    app: path.join(__dirname, 'app'),
    build: path.join(__dirname, 'build')
  }

  const common = {
    entry: {
      app: PATHS.app
    },
    output: {
      path: PATHS.build,
      filename: 'app.js'
    }
  }

  const startConfig = {}

  const buildConfig = {}

  const config = (TARGET === 'start' || !TARGET)
    ? merge(common, startConfig)
    : merge(common, buildConfig)

  export default config
  ```

12. We're going to need React:

  ```sh
  npm i -S react react-dom
  ```

  1. [react](https://github.com/facebook/react) provides the basic React.js library
  2. [react-dom](https://www.npmjs.com/package/react-dom) for working with the DOM

13. Now let's set up our React app. Add the following to `/app/index.jsx`.

  ```jsx
  import React from 'react'
  import { render } from 'react-dom'

  import './main.css'

  import App from './components/app.jsx'

  const div = document.createElement('div')

  document.body.appendChild(div)

  render(<App/>, div)
  ```

  We need React, of course. It also gives us the JSX (Extended JavaScript) capability. As we're working with the DOM (in browserland), we'll need the ReactDOM, too. But we can just grab the one function we need: `render`.

  We use the Webpack `style-loader` to load the styles in our `/app/main.css` file (processing them through the `myth-loader` and `css-loader` first).

  Then we import our App component, create a new HTML div element, append it to the body, and then render our App component into it.

14. Now we can code our App component in `/app/components/app.jsx`.

  ```jsx
  import React from 'react'

  const App = () => <p>Welcome to the App!</p>

  export default App

  ```

  Keep it simple to start. We import React, and build a *presentation* component using a simple ES6 fat arrow function to return our JSX. Then we export it for use in `/app/index.jsx` and elsewhere.

15. We'll want some help with our syntax, so let's install a linter. ESLint is the current favorite. We'll add some plugins and a loader to make it work with Webpack.

  ```sh
  npm i -D eslint eslint-loader \
  eslint-config-standard eslint-config-standard-jsx \
  eslint-config-standard-react eslint-plugin-react \
  eslint-plugin-promise eslint-plugin-standard
  ```

  1. [eslint](http://eslint.org/) checks our JS code to make sure it's correct
  2. [eslint-loader](https://github.com/MoOx/eslint-loader) let's us use ESLint in Webpack
  3. [eslint-config-standard](https://github.com/feross/eslint-config-standard) configures ESLint to use the [standard.js](http://standardjs.com/) style guide&mdash;a nice way to remain consistent
  4. [eslint-config-standard-jsx](https://github.com/feross/eslint-config-standard-jsx) adds Standard.js JSX support
  5. [eslint-config-standard-react](https://github.com/feross/eslint-config-standard-react) adds Standard.js React support
  6. [eslint-plugin-react](https://github.com/yannickcr/eslint-plugin-react) adds React-specific linting rules
  7. [eslint-plugin-promise](https://github.com/xjamundx/eslint-plugin-promise) enforces best practices for JS promises
  8. [eslint-plugin-standard](https://github.com/xjamundx/eslint-plugin-standard) adds some extra rules needed for Standard.js

16. We don't want to lint everything twice, so let's add our `build` folder to `.eslintignore`.

  ```
  build/
  ```

17. We'll also configure ESLint in our `.eslintrc` file. There are a [great many options](http://eslint.org/docs/user-guide/configuring), of course.

  ```
  {
    "extends": [
      "standard",
      "standard-react"
    ],
    "parserOptions": {
      "ecmaVersion": 6,
      "ecmaFeatures": {
        "jsx": true
      },
      "sourceType": "module"
    },
    "env": {
      "browser": true,
      "node": true,
      "mocha": true
    },
    "plugins": [
      "react"
    ]
  }
  ```

18. While we're at it, we can use the `.editorconfig` to [configure our text editors](http://EditorConfig.org) so all our developers are using the same format. These are pretty self-explanatory.

  ```
  # EditorConfig: http://EditorConfig.org

  root = true

  [*]
  indent_style = space
  indent_size = 2

  end_of_line = lf
  charset = utf-8
  trim_trailing_whitespace = true
  insert_final_newline = true

  [*.md]
  trim_trailing_whitespace = false
  ```

19. We can use a `.babelrc` file to configure Babel, or we can just add our `babel` settings to the `package.json` file. See also the [babel-preset-react-hmre](https://github.com/danmartinez101/babel-preset-react-hmre) for details.

  ```json
  "babel": {
    "presets": [
      "es2015",
      "react",
      "stage-1"
    ],
    "env": {
      "start": {
        "presets": [
          "react-hmre"
        ]
      }
    }
  }
  ```

20. Time to set up our tests and add a first specification. We can use `mocha.opts` (in `/test/mocha.opts`) to add arguments that would otherwise need to be included on the command line. Here we require the `setup.js` file before the specs are run, we turn on "full trace" capability, and we tell it to use `babel-register` to transpile the specs from ES6 to ES5 before running them. That allows us to use JavaScript 2015 in our tests. Nice!

  ```
  --require ./test/setup
  --full-trace
  --compilers js:babel-register
  ```

21. Before we run the tests, we want to set up our environment. We'll use [jsdom](https://github.com/tmpvar/jsdom), which is a "JavaScript implementation of the WHATWG DOM and HTML standards, for use with node.js". We'll add the following to our `/test/setup.js` file. I've swiped the [latest recommendation](https://github.com/airbnb/enzyme/blob/master/docs/guides/jsdom.md#using-enzyme-with-jsdom) and updated it slightly to use the spread operator and more ES6:

  ```js
  import { jsdom } from 'jsdom'

  let exposedProperties = [
    'window',
    'navigator',
    'document'
  ]

  global.document = jsdom('')
  global.window = document.defaultView

  // Loop through the defaultView adding properties to exposedProperties
  // and assigning the global property
  Object.keys(document.defaultView).forEach((property) => {
    if (typeof global[property] === 'undefined') {
      exposedProperties = [ ...exposedProperties, property ]
      global[property] = document.defaultView[property]
    }
  })

  global.navigator = {
    userAgent: 'node.js'
  }
  ```

  This creates the most basic HTML DOM and then sets the global variables for the `document`, the `window`, the `navigator`, etc.

22. And we'll need to install all the dependencies we need:

  ```sh
  npm i -D chai chai-enzyme cheerio enzyme jsdom mocha react-addons-test-utils sinon
  ```

  1. [chai](http://chaijs.com/) is a [BDD/TDD](https://www.youtube.com/watch?v=mT8QDNNhExg) assertion library for node and the browser that can be delightfully paired with any javascript testing framework
  2. [chai-enzyme](https://github.com/producthunt/chai-enzyme) adds Chai.js assertions and convenience functions for testing React components with Enzyme
  3. [cheerio](https://github.com/cheeriojs/cheerio) is a fast, flexible, and lean implementation of core jQuery designed specifically for the server and is used by Enzyme for DOM traversal
  4. [enzyme](https://github.com/airbnb/enzyme) is AirBnB's JavaScript testing utilities for React and is easier to use than the Facebook utilities
  5. [jsdom](https://github.com/tmpvar/jsdom) is a JavaScript implementation of the WHATWG DOM and HTML standards, for use with node.js
  6. [mocha](https://mochajs.org/) is a feature-rich JavaScript test framework running on Node.js and in the browser
  7. [react-addons-test-utils](https://facebook.github.io/react/docs/test-utils.html) are the Facebook React test utilities (this is an implicit dependency of enzyme in order to support react@0.13-14)
  8. [sinon](http://sinonjs.org/) provides standalone test spies, stubs and mocks for JavaScript

23. Now we can set up our specifications and add them to the `/test/tests.js` file. The comment at the top prevents ESLint from complaining that `describe` and `it` are not defined in the file.

  ```js
  /*global describe it */

  import React from 'react'

  import chai, { expect } from 'chai'
  import chaiEnzyme from 'chai-enzyme'

  chai.use(chaiEnzyme())

  import { render } from 'enzyme'

  import App from '../app/components/app.jsx'

  describe('<App/>', () => {
    it('displays the welcome message', () => {
      const wrapper = render(<App/>)

      expect(wrapper.text()).to.contain('Welcome')
    })
  })
  ```

  See the [documentation](https://github.com/airbnb/enzyme/blob/master/docs/guides/webpack.md) for more information.

24. Enzyme requires that we add some configuration to our `webpack.config.babel.js` file to work well with React 0.15. Add the following to the `common` config object:

  ```js
  externals: {
    'jsdom': 'window',
    'react/lib/ReactContext': 'window',
    'react/lib/ExecutionEnvironment': true,
    'react/addons': true
  },
  ```

25. We also need the Webpack Dev Server and Hot Module Replacement for our development mode. First we'll add the dependency.

  ```sh
  npm i -D webpack-dev-server
  ```

26. Let's import webpack into our config so we can work with plugins:

  ```js
  import webpack from 'webpack'
  ```

27. And we'll need to update our `webpack.config.babel.js` file to use the module.

  ```js
  const startConfig = {
    devtool: 'eval-source-map',
    devServer: {
      contentBase: PATHS.build,
      historyApiFallback: true,
      hot: true,
      inline: true,
      progress: true,
      stats: 'errors-only',
      host: process.env.HOST,
      port: process.env.PORT
    },
    plugins: [
      new webpack.HotModuleReplacementPlugin()
    ]
  }
  ```

28. Also in `webpack.config.babel.js`, we'll need to configure babel as well by addin the extensions to resolve to the `common` config object (I added it between the `entry` and `output` keys).

  ```js
  resolve: {
    extensions: [ '', '.js', '.jsx' ]
  }
  ```

29. Below the `output` key in the `common` config object we'll add the preloaders (for linting) and the loaders in a `module` block.

  ```js
  module: {
    preLoaders: [
      {
        test: /\.jsx?$/,
        loaders: [ 'eslint' ],
        include: PATHS.app
      }
    ],
    loaders: [
      {
        test: /\.jsx?$/,
        loaders: [ 'babel?cacheDirectory' ],
        include: PATHS.app
      }
    ]
  }
  ```

  Preloaders run first, then loaders. Each load step can use one or more loaders. The `test` key provides a regular expression to test file names against. This regular expression matches all filenames that *end* with either `.js` or `.jsx`. In the preload stage, the `eslint-loader` is run on all files in the `PATHS.app` path ending with `.js` or `.jsx`. (You can leave off the `-loader` part.)

  In the loading phase, the `babel-loader` will be run against those same files, and we've told it to cache the results so that we don't rerun the babel compiler needlessly.

30. While we're at it, let's extend it with the CSS loaders and preLoaders, too. And we'll add a `postcss` block to configure the `stylelint` linter for CSS. Here we set it to insist on lowercase letters when colors are specified in Hex. Dang if that don't look better. Add the import to the imports at the top of the file.

  ```js
  import stylelint from 'stylelint'

  module: {
    preLoaders: [
      {
        test: /\.css$/,
        loaders: [ 'postcss' ],
        include: PATHS.app
      },
      {
        test: /\.jsx?$/,
        loaders: [ 'eslint' ],
        include: PATHS.app
      }
    ],
    loaders: [
      {
        test: /\.css$/,
        loaders: [ 'style', 'css', 'myth' ],
        include: PATHS.app
      },
      {
        test: /\.jsx?$/,
        loaders: [ 'babel?cacheDirectory' ],
        include: PATHS.app
      }
    ]
  },
  postcss: function () {
    return [stylelint({
      rules: {
        'color-hex-case': 'lower'
      }
    })]
  }
  ```

  These work the same way. The test regex matches filenames ending in `.css` in the `PATHS.app` path. Then it applies the loader(s). In the preload stage, `postcss` is used to run the `stylelint`. This checks our CSS for errors. We configure `postcss` in the `postcss` function below, passing a rule to `stylelint` (as an example) to warn us of any hex color values not using lowercase letters.

  In the load step, three loaders are applied from right to left. First the `myth-loader` converts future CSS syntax into current syntax. Then the `css-loader` pulls in any imports, etc. and outputs the final CSS. Finally, the `style-loader` inserts the CSS into our bundled code.

31. We'll need to add the loader and lint dependencies:

  ```sh
  npm i -D style-loader css-loader myth-loader stylelint postcss-loader
  ```

  1. [style-loader](https://github.com/webpack/style-loader) adds the CSS to the exported DOM in a `<style>` element
  2. [css-loader](https://github.com/webpack/css-loader) loads CSS file(s) with resolved imports and returns CSS code
  3. [myth-loader](https://github.com/besarthoxhaj/myth-loader) converts future CSS syntax to current syntax with [myth.io](http://www.myth.io/)
  4. [stylelint](https://github.com/stylelint/stylelint) is a "mighty, modern" [CSS linter](http://stylelint.io/)
  5. [postcss-loader](https://github.com/postcss/postcss-loader) postprocesses your CSS with [PostCSS](http://postcss.org/) plugins (we could also have used the [stylelint-loader](https://www.npmjs.com/package/stylelint-loader) directly, but I thought it would be nice to see some postcss as well)

32. We need the `hmre` plugin, too.

  ```sh
  npm i -D babel-preset-react-hmre
  ```

  [babel-preset-react-hmre](https://github.com/danmartinez101/babel-preset-react-hmre): This preset will configure Babel 6 for [HMR](https://github.com/gaearon/react-transform-hmr) and friends. It is recommended that this preset only be configured for your development builds.

33. Confused as to what our `webpack.config.babel.js` file should look like now? Well, here it is in full:

  ```js
  import webpack from 'webpack'
  import merge from 'webpack-merge'
  import path from 'path'
  import stylelint from 'stylelint'

  const TARGET = process.env.npm_lifecycle_event

  process.env.BABEL_ENV = TARGET

  const PATHS = {
    app: path.join(__dirname, 'app'),
    build: path.join(__dirname, 'build')
  }

  const common = {
    entry: {
      app: PATHS.app
    },
    resolve: {
      extensions: [ '', '.js', '.jsx' ]
    },
    output: {
      path: PATHS.build,
      filename: 'app.js'
    },
    externals: {
      'jsdom': 'window',
      'react/lib/ReactContext': 'window',
      'react/lib/ExecutionEnvironment': true,
      'react/addons': true
    },
    module: {
      preLoaders: [
        {
          test: /\.css$/,
          loaders: [ 'postcss' ],
          include: PATHS.app
        },
        {
          test: /\.jsx?$/,
          loaders: [ 'eslint' ],
          include: PATHS.app
        }
      ],
      loaders: [
        {
          test: /\.css$/,
          loaders: [ 'style', 'css', 'myth' ],
          include: PATHS.app
        },
        {
          test: /\.jsx?$/,
          loaders: [ 'babel?cacheDirectory' ],
          include: PATHS.app
        }
      ]
    },
    postcss: function () {
      return [stylelint({
        rules: {
          'color-hex-case': 'lower'
        }
      })]
    }
  }

  const startConfig = {
    devtool: 'eval-source-map',
    devServer: {
      contentBase: PATHS.build,
      historyApiFallback: true,
      hot: true,
      inline: true,
      progress: true,
      stats: 'errors-only',
      host: process.env.HOST,
      port: process.env.PORT
    },
    plugins: [
      new webpack.HotModuleReplacementPlugin()
    ]
  }

  const buildConfig = {}

  const config = (TARGET === 'start' || !TARGET)
    ? merge(common, startConfig)
    : merge(common, buildConfig)

  export default config
  ```

34. We need to add some scripts to our `package.json` file to run building, linting, and testing tasks, and for running the development server in HMR mode:

  ```js
  "build": "webpack",
  "lint": "eslint . --ext .js --ext .jsx --cache || true",
  "start": "webpack-dev-server",
  "test": "mocha --opts ./test/mocha.opts test/tests.js"
  ```

  - Use `npm run build` to compile and bundle the source code into `/build/app.js`
  - Use `npm run lint` to lint the source code and report warnings and errors
  - Use `npm start` to start the app with `webpack-dev-server`
  - Use `npm test` to run the specifications in `/test/tests.js`

35. Now we can run the Webpack Dev Server.

  ```sh
  npm start
  ```

  You should be able to see the app (just a line in red that says, "Welcome to the App!") at [http://localhost:8080/](http://localhost:8080/). If we make changes to a source file and then save them, the app should reload instantly. Try it. Change "App!" to "Flapp!" or something like that, save the file, and check that the page has reloaded with the new text.

36. Now let's add [react-bootstrap](https://react-bootstrap.github.io/) and have some fun:

  ```sh
  npm i -S react-bootstrap
  ```

37. We can add a header. Create `/app/components/header.jsx` and add:

  ```jsx
  import React from 'react'

  import { Nav, Navbar, NavItem } from 'react-bootstrap'

  const Header = () => <Navbar>
    <Navbar.Header>
      <Navbar.Brand>
        <a href='#'>Didactic Doodle</a>
      </Navbar.Brand>
    </Navbar.Header>
    <Nav>
      <NavItem eventKey={1} href='#'>Home</NavItem>
      <NavItem eventKey={2} href='#'>About</NavItem>
    </Nav>
  </Navbar>

  export default Header
  ```

  Whoa! Where did all this come from? Simple. I just went to the react-bootstrap [Navbar Basic Example](https://react-bootstrap.github.io/components.html#navbars-basic), clicked on "show code", and swiped the JSX code. Then I dropped the dropdown (heh) and modified the code as necessary. ESLint warned me which imports I was missing so I added them up top. The links won't work, but we have a navbar at least and that's a start. We got a lot of benefit for just a few seconds work.

38. Now let's use our header in `/app/components/app.jsx`. We'll take advantage of react-bootstrap's `Grid` component as well (this just adds a `<div class="container"></div>` wrapper).

  ```jsx
  import React from 'react'

  import { Grid } from 'react-bootstrap'

  import Header from './header.jsx'

  const App = () => <div>
    <Header/>
    <Grid><p>Welcome to the Zapp!</p></Grid>
  </div>

  export default App
  ```

  Take a look at our app now and you should see a nice navbar at the top. That was easy!

39. Now, let's add some pages:

  ```jsx
  // /app/components/home.jsx
  import React from 'react'

  import { Col, Row } from 'react-bootstrap'

  const Home = () => <Row>
    <Col xs={12}>
      <h1>Home</h1>
    </Col>
  </Row>

  export default Home
  ```

  ```jsx
  // /app/components/about.jsx
  import React from 'react'

  import { Col, Row } from 'react-bootstrap'

  const About = () => <Row>
    <Col xs={12}>
      <h1>About</h1>
    </Col>
  </Row>

  export default About
  ```

40. We're going to need some kind of router to allow us to navigate to our new pages. To begin, let's add `react-router` (we'll also need `react-router-bootstrap` to help integrate `react-router` and `react-bootstrap`).

  ```sh
  npm i -S react-router react-router-bootstrap
  ```

41. Now we can update our `Header` to make the links work. We'll need to update the regular links to make them work with `react-router`. We can do this by wrapping them in the appropriate containers provided by `react-router-bootstrap`:

  ```jsx
  import React from 'react'

  import { Nav, Navbar, NavItem } from 'react-bootstrap'

  import { IndexLinkContainer, LinkContainer } from 'react-router-bootstrap'

  const Header = () => <Navbar>
    <Navbar.Header>
      <Navbar.Brand>
        <IndexLinkContainer to={{ pathname: '/' }}>
          <a>Setup</a>
        </IndexLinkContainer>
      </Navbar.Brand>
      <Navbar.Toggle />
    </Navbar.Header>
    <Navbar.Collapse>
      <Nav>
        <IndexLinkContainer to={{ pathname: '/' }}>
          <NavItem eventKey={1} href='#'>Home</NavItem>
        </IndexLinkContainer>
        <LinkContainer to={{ pathname: '/about' }}>
          <NavItem eventKey={2} href='#'>About</NavItem>
        </LinkContainer>
      </Nav>
    </Navbar.Collapse>
  </Navbar>

  export default Header
  ```

42. We'll need to update `/app/index.jsx` to add our routes.

  ```jsx
  import React from 'react'
  import { render } from 'react-dom'
  import { browserHistory, IndexRoute, Route, Router } from 'react-router'

  import './main.css'

  import App from './components/app.jsx'
  import Home from './components/home.jsx'
  import About from './components/about.jsx'

  const div = document.createElement('div')

  document.body.appendChild(div)

  render(<Router history={browserHistory}>
    <Route path='/' component={App}>
      <IndexRoute component={Home}/>
      <Route path='about' component={About}/>
    </Route>
  </Router>, div)
  ```

  1. `browserHistory` allows us to use pushState to control the browser history (we'll need to take this into account on the back end so that our routes don't get 404s if the user reloads a page)
  2. `IndexRoute` is used for the default page
  3. `Route` adds a route, and
  4. `Router` collects the routes and provides the `react-router` functionality

  This should be pretty easy to understand if you've worked with routing before.

43. Now we should be able to switch between pages, but if we try it we won't see any difference. That's because the individual pages (home or about) are being passed to `App` as children, but our current `App` doesn't do anything with them. So let's update our `/app/components/app.jsx` file accordingly.

  ```js
  import React, { PropTypes } from 'react'

  import { Grid } from 'react-bootstrap'

  import Header from './header.jsx'

  const App = ({ children }) => <div>
    <Header/>
    <Grid>{children}</Grid>
  </div>

  App.propTypes = {
    children: PropTypes.node.isRequired
  }

  export default App
  ```

  We import the PropTypes from React so we can specify the type of `children` (an object). The first argument to our `App` function is the `props`. We use destructuring assignment here to pull out the `children` prop, then we insert the value of `children` into the `<Grid>` component. Finally, we set the `propTypes` for the `App` component and indicate that the `children` prop is required.

  If you run the app and try the links now, you should see the page change. We now have a single-page app.

44. We've broken our App spec now. Let's rewrite it to check that the Header is present instead:

  ```js
  /*global describe it */

  import React from 'react'

  import chai, { expect } from 'chai'
  import chaiEnzyme from 'chai-enzyme'

  chai.use(chaiEnzyme())

  import { shallow } from 'enzyme'

  import App from '../app/components/app.jsx'
  import Header from '../app/components/header.jsx'

  describe('<App/>', () => {
    it('includes the <Header/>', () => {
      const wrapper = shallow(<App><p/></App>)

      expect(wrapper).to.contain(<Header/>)
    })
  })
  ```

  We've switched from `render` to `shallow`. We need to pass a child to `<App>` (it's expecting a route, but we're doing a shallow render here and so we'll just send an empty paragraph). Finally, we check that the shallowly rendered `<App/>` component includes a `<Header/>`. If you run it with `npm test` it should work.

44. Let's add redux to manage state, and we'll use `react-redux-router` to store our current route state in the redux store as well.

  ```sh
  npm i -S redux react-redux react-router-redux
  ```

45. Just to give us something to play with, we'll add a simple reducer that takes a single value and increments or decrements it. Create `app/reducer.js` and add:

  ```js
  const INCREMENT = 'INCREMENT'
  const DECREMENT = 'DECREMENT'

  const reducer = (state = { count: 0 }, action) => {
    switch (action.type) {
      case INCREMENT:
        return {
          ...state,
          count: state.count + 1
        }
      case DECREMENT:
        return {
          ...state,
          count: state.count - 1
        }
      default:
        return state
    }
  }

  export default reducer
  ```

  Our reducer takes the current state and an action and returns a new state. Here we have two action types: increment and decrement, which do exactly what you'd expect. If some other action is passed (or none), we return the state unchanged. Our state is just an object with a single key, `count` and an integer value, defaulting to 0.

  Later we'll add action creators, containers, etc., but for now this is enough to get started.

46. In our top level `/app/index.jsx` file, we'll create the store from our reducer. Then we'll use a `Provider` wrapper (which we get from `react-redux`) to pass the store into the React context. We can then pull it out of the context where we need it.

  ```js
  import React from 'react'
  import { render } from 'react-dom'
  import { browserHistory, IndexRoute, Route, Router } from 'react-router'
  import { createStore } from 'redux'
  import { Provider } from 'react-redux'

  import './main.css'

  import App from './components/app.jsx'
  import Home from './components/home.jsx'
  import About from './components/about.jsx'

  import reducer from './reducer.js'

  const store = createStore(reducer)

  const div = document.createElement('div')

  document.body.appendChild(div)

  render(<Provider store={store}>
    <Router history={browserHistory}>
      <Route path='/' component={App}>
        <IndexRoute component={Home}/>
        <Route path='about' component={About}/>
      </Route>
    </Router>
  </Provider>, div)
  ```

47. We'll want to see what's going on with our store, so let's add the Redux tools, `redux-devtools`, `redux-devtools-dock-monitor`, and `redux-devtools-log-monitor`:

  ```sh
  npm i -D redux-devtools redux-devtools-dock-monitor redux-devtools-log-monitor
  ```

  1. [redux-devtools](https://github.com/gaearon/redux-devtools) with hot reloading, action replay, and [customizable UI](http://youtube.com/watch?v=xsSnOQynTHs)
  2. [redux-devtools-dock-monitor](https://github.com/gaearon/redux-devtools-dock-monitor) a resizable and movable dock for Redux DevTools monitors
  3. [redux-devtools-log-monitor](https://github.com/gaearon/redux-devtools-log-monitor) the default monitor for Redux DevTools with a tree view

48. We'll add them to our `/app/index.jsx` file thus:

  ```jsx
  import { createDevTools } from 'redux-devtools'
  import LogMonitor from 'redux-devtools-log-monitor'
  import DockMonitor from 'redux-devtools-dock-monitor'

  import React from 'react'
  import { render } from 'react-dom'
  import { browserHistory, IndexRoute, Route, Router } from 'react-router'
  import { createStore } from 'redux'
  import { Provider } from 'react-redux'

  import './main.css'

  import App from './components/app.jsx'
  import Home from './components/home.jsx'
  import About from './components/about.jsx'

  import reducer from './reducer.js'

  const DevTools = createDevTools(
    <DockMonitor toggleVisibilityKey='ctrl-h' changePositionKey='ctrl-q'>
      <LogMonitor theme='tomorrow' preserveScrollTop={false} />
    </DockMonitor>
  )

  const store = createStore(
    reducer,
    DevTools.instrument()
  )

  const div = document.createElement('div')

  document.body.appendChild(div)

  render(<Provider store={store}>
    <div>
      <Router history={browserHistory}>
        <Route path='/' component={App}>
          <IndexRoute component={Home}/>
          <Route path='about' component={About}/>
        </Route>
      </Router>
      <DevTools/>
    </div>
  </Provider>, div)
  ```

  Now we should be able to see the dock. Try it. You can hide and show the dock with Control-h. You can change where it appears on the screen with Control-q.

49. Things are looking good. But we'd like to store our current routing state in our Redux store as well. We'll begin by importing the tools we need. So replace this:

  ```js
  import { createStore } from 'redux'
  ```

  with this:

  ```js
  import { combineReducers, createStore } from 'redux'
  import { routerReducer, syncHistoryWithStore } from 'react-router-redux'
  ```

50. Next, we'll add the `routerReducer` to our regular reducer using `combineReducers`. Replace this:

  ```js
  const store = createStore(
    reducer,
    DevTools.instrument()
  )
  ```

  with this:

  ```js
  const reducers = combineReducers({
    reducer,
    routing: routerReducer
  })

  const store = createStore(
    reducers,
    DevTools.instrument()
  )
  ```

  It should be pretty obvious what this does.

51. Finally, we'll synchronize our browser history with the store. Replace this:

  ```js
  render(<Provider store={store}>
    <div>
      <Router history={browserHistory}>
        <Route path='/' component={App}>
          <IndexRoute component={Home}/>
          <Route path='about' component={About}/>
        </Route>
      </Router>
      <DevTools/>
    </div>
  </Provider>, div)
  ```

  with this:

  ```js
  const history = syncHistoryWithStore(browserHistory, store)

  render(<Provider store={store}>
    <div>
      <Router history={history}>
        <Route path='/' component={App}>
          <IndexRoute component={Home}/>
          <Route path='about' component={About}/>
        </Route>
      </Router>
      <DevTools/>
    </div>
  </Provider>, div)
  ```

  Start the app with `npm start` and load the page at [http://localhost:8080/](http://localhost:8080/), then use Control-h, if necessary, to show the Redux Dock. Switch back and forth between the Home and About pages using the navbar, and see how the store changes. Look under the `router` key.

52. Let's also add some tests to test the reducer. In `/test/tests.js` we'll import the reducer and `createStore` from `redux`:

  ```js
  import { createStore } from 'redux'

  import reducer from '../app/reducer.js'
  ```

  Then we'll add the tests:

  ```js
  describe('reducer', () => {
    it('increments the count when an INCREMENT action is dispatched', () => {
      const store = createStore(reducer)
      const action = { type: 'INCREMENT' }

      store.dispatch(action)
      store.dispatch(action)
      store.dispatch(action)

      expect(store.getState().count).to.equal(3)
    })

    it('increments the count when an DECREMENT action is dispatched', () => {
      const store = createStore(reducer)
      const action = { type: 'DECREMENT' }

      store.dispatch(action)
      store.dispatch(action)
      store.dispatch(action)

      expect(store.getState().count).to.equal(-3)
    })
  })
  ```

  Now if you run `npm test` you should see three passing tests.
