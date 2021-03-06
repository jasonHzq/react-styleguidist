# Installation

**Requirements:** Only Babel 6 is supported in [Styleguidist 2.0.0](https://github.com/sapegin/react-styleguidist/releases/tag/2.0.0)+. If you don't use Babel in your project, that's fine, but if you use Babel 5, please use Styleguidist 1.3.2. Webpack is recommended, but not required. Should work with Webpack 1 and Webpack 2.

1. Install from npm:

   ```bash
   npm install --save-dev react-styleguidist
   ```

2. Add a **`styleguide.config.js`** file into your project’s root folder. A simplest possible config looks like this:

   ```javascript
   module.exports = {
     title: 'My Great Style Guide',
     components: './lib/components/**/*.js',
   };
  ```

3. If you use transpilers to run your project files (JSX → JS, SCSS → CSS, etc.), you need to set them up for the style guide too.

   Styleguidist generates a Webpack config required for the style guide itself but you need to configure [Webpack loaders](https://webpack.github.io/docs/configuration.html#module-loaders) for your project.

   Add the `updateWebpackConfig` function to your `styleguide.config.js`:

   ```javascript
   var path = require('path');
   module.exports = {
     updateWebpackConfig: function(webpackConfig, env) {
       // Your source files folder or array of folders, should not include node_modules
       let dir = path.join(__dirname, 'src');
       webpackConfig.module.loaders.push(
         // Babel loader will use your project’s .babelrc
         {
           test: /\.jsx?$/,
           include: dir,
           loader: 'babel'
         },
         // Other loaders that is needed for your components
         {
           test: /\.css$/,
           include: dir,
           loader: 'style!css?modules&importLoaders=1'
         }
       );
       return webpackConfig;
     },
   };
   ```

   **Note**: don’t forget `include` or `exclude` options for your Webpack loaders, otherwise they will interfere with Styleguidist’s loaders. Also do not include `node_modules` folder.

4. Configure [hot module replacement](https://github.com/gaearon/react-transform-hmr). This is optional, but highly recommended.

   Install React Transform for Babel, if you don’t have it yet: `npm install --save-dev babel-preset-react-hmre`.

   When you run the Styleguidist server, `NODE_ENV` is set to `development` and when you build style guide `NODE_ENV` is set to `production`, so you can enable hot module replacement only in development. Update your `.babelrc`:

   ```json
   {
     "presets": ["es2015", "react"],
     "env": {
       "development": {
         "presets": ["react-hmre"]
       }
     }
   }
   ```

5. Add these scripts to your `package.json`:

  ```javascript
  {
    // ...
    "scripts": {
      "styleguide-server": "styleguidist server",
      "styleguide-build": "styleguidist build"
    }
  }
  ```

6. Run **`npm run styleguide-server`** to start style guide dev server.

To customize your style guide, head to the [Configuration section](Configuration.md).


# Documenting components

Styleguidist generates documentation from three sources:

* **PropTypes and component description** in the source code

  Components’ `PropTypes` and documentation comments are parsed by the [react-docgen](https://github.com/reactjs/react-docgen) library. Have a look at [their example](https://github.com/reactjs/react-docgen#example) of a component documentation. You can change its behaviour using `propsParser` and `resolver` [options](Configuration.md).

  [Flow](https://flowtype.org/) type annotations are supported too.

* **Usage examples and further documentation** in Markdown

  Examples are written in Markdown where any code block without a language tag will be rendered as a React component. By default any `Readme.md` in the component’s folder is treated as an examples file but you can change it with the `getExampleFilename` [option](Configuration.md).

      React component example:

          <Button size="large">Push Me</Button>

      One more with generic code fence:

      ```
      <Button size="large">Push Me</Button>
      ```

      One more with `example` code fence (text editors may alias to `jsx` or `javascript`):
    
      ```example
      <Button size="large">Push Me</Button>
      ```

      This example rendered only as highlighted source code:

      ```html
      <Button size="large">Push Me</Button>
      ```

      Any [Markdown](http://daringfireball.net/projects/markdown/) is **allowed** _here_.

* **External examples using doclet tags**

  Additional example files can be associated with components using doclet (`@example`) syntax. The following component will also have an example as loaded from the `extra.examples.md` file:

  ```javascript
  /**
   * Component is described here.
   *
   * @example ./extra.examples.md
   */
  export default class SomeComponent extends React.Component {
    // ...
  }
  ```

# Writing code examples

Code examples in Markdown use the ES6+JSX syntax. They can access all the components of your style guide using global variables.

You can also `require` other modules (e.g. mock data that you use in your unit tests) from examples in Markdown:

```javascript
const mockData = require('./mocks');
<Message content={mockData.hello} />
```

As an utility, also the [Lodash](https://lodash.com/) library is available globally as `_`.

Each example has its own state that you can access at the `state` variable and change with the `setState` function. Default state is `{}`.

```html
<div>
  <button onClick={() => setState({isOpen: true})}>Open</button>
  <Modal isOpen={state.isOpen}>
    <h1>Hallo!</h1>
    <button onClick={() => setState({isOpen: false})}>Close</button>
  </Modal>
</div>
```

If you want to set the default state you can do:

```javascript
initialState = { key: 42 };
```

You *can* use `React.createClass` in your code examples, but if you need a more complex demo it’s often a good idea to define it in a separate JavaScript file instead and then just `require` it in Markdown.
