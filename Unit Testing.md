## Unit Testing

Unit testing is an excellent approach to protect your code against problems before deploying it. While Gatsby does not contain support for unit testing out of the box, getting started is simple. However, there are a few aspects of the Gatsby build process that make the usual Jest configuration ineffective.
## Setting up the environment
Jest, which was built by Facebook, is the most popular testing framework for React. While Jest is a general-purpose JavaScript unit testing framework, it has many characteristics that make it particularly well suited for use with React.
### 1. Installing dependencies

First, install Jest and some more required packages. Then babel-jest and babel-present-gatsby to to ensure that the babel preset(s) used correspond to those used internally by your Gatsby site.

npm install --save-dev jest babel-jest react-test-renderer babel-preset-gatsby identity-obj-proxy

### 2. Creating a configuration file for Jest
jest.config.js

module.exports = {

`    `transform: {

`      `"^.+\\.jsx?$": `<rootDir>/jest-preprocess.js`,

`    `},

`    `moduleNameMapper: {

`      `".+\\.(css|styl|less|sass|scss)$": `identity-obj-proxy`,

`      `".+\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": `<rootDir>/\_\_mocks\_\_/file-mock.js`,

`      `"^gatsby-page-utils/(.\*)$": `gatsby-page-utils/dist/$1`, 

`      `"^gatsby-core-utils/(.\*)$": `gatsby-core-utils/dist/$1`, 

`      `"^gatsby-plugin-utils/(.\*)$": [

`        ``gatsby-plugin-utils/dist/$1`,

`        ``gatsby-plugin-utils/$1`,

`      `],

`    `},

`    `testPathIgnorePatterns: [`node\_modules`, `\\.cache`, `<rootDir>.\*/public`],

`    `transformIgnorePatterns: [`node\_modules/(?!(gatsby|gatsby-script|gatsby-link)/)`],

`    `globals: {

`      `\_\_PATH\_PREFIX\_\_: ``,

`    `},

`    `testEnvironmentOptions: {

`      `url: `http://localhost`, },

`    `setupFiles: [`<rootDir>/loadershim.js`

],

}

Going over the content of this configuration file:

The transform section instructs Jest that all js or jsx files in the project root must be transformed using a jest-preprocess.js file  where we configure Babel with the following simple configuration:

jest-preprocess.js

const babelOptions = {

`    `presets: ["babel-preset-gatsby"],

`  `}



`  `module.exports = require("babel-jest").default.createTransformer(babelOptions)

The next option is moduleNameMapper which functions similarly to webpack rules in that it instructs Jest on how to handle imports. Here, we are primarily interested with imitating static file imports, which Jest is incapable of handling. A mock is a dummy module that is used in testing instead of the real module. It's useful when you can't or don't want to test anything. Mocking anything is possible, and in this case, we are mocking assets rather than code. You must use the identity-obj-proxy package for stylesheets. You must use a manual mock called file-mock.js for any other assets. You must make this yourself. For this, the norm is to create a directory called \_\_mocks\_\_ in the root directory.

\_\_mocks\_\_/file-mock.js 

module.exports = "test-file-stub"

The following configuration option is testPathIgnorePatterns where we are instructing Jest to ignore any tests found in the node modules or.cache directories.

The next option is critical and differs from what is found in other Jest manuals. Because Gatsby includes un-transpiled ES6 code, we'll need transformIgnorePatterns. Jest does not try to translate code inside node modules by default, thus we will get an error like this:

TEXT

**/my-app/node\_modules/gatsby/cache-dir/gatsby-browser-entry.js:1**

**({"Object.<anonymous>":function(module,exports,require,\_\_dirname,\_\_filename,global,jest){import React from "react"**

`                                                                                            `**^^^^^^**

**SyntaxError: Unexpected token import**

This is due to the fact that gatsby-browser-entry.js is not transpiled before executing in Jest. This can be resolved by setting the default transformIgnorePatterns to exclude the gatsby module.

- The globals section configures \_\_PATH PREFIX\_\_, which is normally established by Gatsby and is required by various components.

- Because some DOM APIs, such as localStorage, are unhappy with the default, you must set testURL to a real URL (about:blank).


- There's one more global to configure, but because it's a function, and we can't do it in the JSON. The setupFiles array is ideal for this since it allows us to specify which files will be included before all tests are run.

loadershim.js

global.\_\_\_loader = {

`    `enqueue: jest.fn(),

`  `}

### 3. Useful mocks to complete your testing environment 

#### Mocking **Gatsby**
Finally, mocking the Gatsby module is a good idea. This may not be necessary at initially, but it will make testing components that utilise Link or GraphQL much easier.

\_\_mocks\_\_/gatsby.js 

const React = require("react")

const gatsby = jest.requireActual("gatsby")

module.exports = {

`  `...gatsby,

`  `graphql: jest.fn(),

`  `Link: jest.fn().mockImplementation(

`    `// these props are invalid for an `a` tag

`    `({

`      `activeClassName,

`      `activeStyle,

`      `getProps,

`      `innerRef,

`      `partiallyActive,

`      `ref,

`      `replace,

`      `to,

`      `...rest

`    `}) =>

`      `React.createElement("a", {

`        `...rest,

`        `href: to,

`      `})

`  `),

`  `StaticQuery: jest.fn(),

`  `useStaticQuery: jest.fn(),

}

This mocks the graphql() function, as well as the Link and StaticQuery components.

### 4. Writing Tests

This manual provides just a snapshot test to check if everything is working:

Next we create  the test file first which can be placed in a \_\_tests\_\_ directory or elsewhere with the extension.spec.js or.test.js.

The \_\_tests\_\_ folder convention will be used in this guide. Then, a header.js file in src/components/ tests / to test the header component :

src/components/\_\_tests\_\_/header.js

import React from "react"

import renderer from "react-test-renderer"

import Header from "../header"

describe("Header", () => {

`  `it("renders correctly", () => {

`    `const tree = renderer

`      `.create(<Header siteTitle="Default Starter" />)

`      `.toJSON()

`    `expect(tree).toMatchSnapshot()

`  `})

})


This is a very brief snapshot test that renders the component with react-test-renderer and then generates a snapshot of it on the first run. It then compares future snapshots to this, allowing you to easily check for regressions. Visit the Jest documentation to learn more about various types of tests you can write.

### 5. Running tests

In package.json, there is a script for test which results in an error, hence, we change to :

package.json 

"scripts": {

`  `"test": "jest"

}

We can now run the tests by ‘npm test’.  It should work now.
###
### 6. Using TypeScript
###
### We plan to use TypeScript in our website for which we need to install typings packages 
npm install --save-dev @types/jest @types/react-test-renderer @babel/preset-typescript
###
### and make the following two changes:
1. ### Updating transform in jest.config.js to run jest-preprocess on files in project’s root directory

jest.config.js 

"^.+\\.[jt]sx?$": "<rootDir>/jest-preprocess.js",

1. Updating jest-preprocess.js with:

jest-preprocess.js

const babelOptions = {

`  `presets: ["babel-preset-gatsby", "@babel/preset-typescript"],

}


More information is at: <https://jestjs.io/docs/getting-started>

<https://www.gatsbyjs.com/docs/how-to/testing/unit-testing/#2-creating-a-configuration-file-for-jest>
