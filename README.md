# Webpacker
![travis-ci status](https://api.travis-ci.org/rails/webpacker.svg?branch=master)

Webpacker makes it easy to use the JavaScript preprocessor and bundler [Webpack](https://webpack.github.io)
to manage application-like JavaScript in Rails. It coexists with the asset pipeline,
as the purpose is only to use Webpack for app-like JavaScript, not images, css, or
even JavaScript Sprinkles (that all continues to live in app/assets).

It's designed to work with Rails 5.1+ and makes use of the [Yarn](https://yarnpkg.com) dependency management
that's been made default from that version forward.

## Installation

Webpacker is currently compatible with Rails 4.2+, but there's no guarantee it will still be
in the future.

You can either make use of Webpacker during setup of a new application with `--webpack`
or you can add the gem and run `./bin/rails webpacker:install` in an existing application.

As the rubygems version isn't promised to be kept up to date until the release of Rails 5.1, you may want to include the gem directly from GitHub:

```ruby
gem 'webpacker', github: 'rails/webpacker'
```

You can also see a list of available commands by running `./bin/rails webpacker`

## Binstubs

Webpacker ships with three binstubs: `./bin/webpack`, `./bin/webpack-watcher` and `./bin/webpack-dev-server`.
They're thin wrappers around the standard webpack.js executable, just to ensure that the right configuration
file is loaded.


A binstub is also created to install your npm dependencies,
and can be called via `./bin/yarn`.

In development, you'll need to run `./bin/webpack-watcher` in a separate terminal from
`./bin/rails server` to have your `app/javascript/packs/*.js` files compiled as you make changes.
If you'd rather not have to run the two processes separately by hand, you can use
[Foreman](https://ddollar.github.io/foreman).

Alternatively, you can run `./bin/webpack-dev-server`. This will launch a
[Webpack Dev Server](https://webpack.github.io/docs/webpack-dev-server.html) listening on http://localhost:8080/
serving your pack files. This setup allows you to leverage advanced Webpack features, such
as [Hot Module Replacement](https://webpack.github.io/docs/hot-module-replacement-with-webpack.html).


## Configuration

Webpacker gives you a default set of configuration files for development and production. They
all live together with the shared points in `config/webpack/*.js`. By default, you shouldn't have to
make any changes for a basic setup out the box. But this is where you do go if you need something
more advanced.

The configuration for what Webpack is supposed to compile by default rests on the convention that
every file in `app/javascript/packs/*` should be turned into their own output files (or entry points,
as Webpack calls it).

Let's say you're building a calendar. Your structure could look like this:

```js
// app/javascript/packs/calendar.js
require('calendar')
```

```
app/javascript/calendar/index.js // gets loaded by require('calendar')
app/javascript/calendar/components/grid.jsx
app/javascript/calendar/styles/grid.sass
app/javascript/calendar/models/month.js
```

```erb
<%# app/views/layout/application.html.erb %>
<%= javascript_pack_tag 'calendar' %>
<%= stylesheet_pack_tag 'calendar' %>
```

But it could also look a million other ways.

## Advanced Configuration

By default, webpacker offers simple conventions for where the webpack configs, javascript app files and compiled webpack bundles will go in your rails app,
but all these options are configurable from `config/webpack/paths.yml` file.

```yml
# config/webpack/paths.yml
source: app/javascript
entry: packs
output: public
config: config/webpack
node_modules: node_modules
```

*Note:* Behind the scenes, webpacker will use same `entry` directory name inside `output`
directory to emit bundles. For ex, `public/packs`

Similary, you can also control and configure `webpack-dev-server` settings from
`config/webpack/development.server.yml` file

```yml
# config/webpack/development.server.yml
enabled: true
host: localhost
port: 8080
```

By default, `webpack-dev-server` uses `output` option specified in
`paths.yml` as `contentBase`.

**Note:** Don't forget to disable `webpack-dev-server` incase you are using
`./bin/webpack-watcher` to serve assets in development mode otherwise
you will get 404 for assets because the helper tag will use webpack-dev-server url
to serve assets instead of public directory.

## Linking to static assets

Static assets like images, fonts and stylesheets support is enabled out-of-box so, you can link them into your javascript app code and have them compiled automatically.

```js
// React component example
// app/javascripts/packs/hello_react.jsx
import React from 'react'
import ReactDOM from 'react-dom'
import helloIcon from '../hello_react/images/icon.png'
import './hello-react.sass'

const Hello = props => (
  <div className="hello-react">
    <img src={helloIcon} alt="hello-icon" />
    <p>Hello {props.name}!</p>
  </div>
)
```

under the hood webpack uses [extract-text-webpack-plugin](https://github.com/webpack-contrib/extract-text-webpack-plugin) plugin to extract all the referenced styles and compile it into a separate `[pack_name].css` bundle so that within your view you can use the `stylesheet_pack_tag` helper,

```erb
<%= stylesheet_pack_tag 'hello_react' %>
```

## Getting asset path

Webpacker provides `asset_pack_path` helper to get the path of any given asset that's been compiled by webpack.

**For ex,** if you want to create a `<link rel="prefetch">` or `<img />`
for an asset used in your pack code you can reference them like this in your view,

```erb
<%= asset_pack_path 'hello_react.css' %>
<% # => "/packs/hello_react.css" %>
<img src="<%= asset_pack_path 'calendar.png' %>" />
<% # => <img src="/packs/calendar.png" /> %>
```

## Deployment

Webpacker hooks up a new `webpacker:compile` task to `assets:precompile`, which gets run whenever you run `assets:precompile`. The `javascript_pack_tag` and `stylesheet_pack_tag` helper method will automatically insert the correct HTML tag for compiled pack. Just like the asset pipeline does it. By default the output will look like this in different environments,

```html
  <!-- In development mode with webpack-dev-server -->
  <script src="http://localhost:8080/calendar.js"></script>
  <link rel="stylesheet" media="screen" href="http://localhost:8080/calendar.css">
  <!-- In development mode -->
  <script src="/packs/calendar.js"></script>
  <link rel="stylesheet" media="screen" href="/packs/calendar.css">
  <!-- In production mode -->
  <script src="/packs/calendar-0bd141f6d9360cf4a7f5.js"></script>
  <link rel="stylesheet" media="screen" href="/packs/calendar-dc02976b5f94b507e3b6.css">
```

## Linking to sprockets assets

It's possible to link to assets that have been precompiled by sprockets. Add the `.erb` extension
to your javascript file, then you can use Sprockets' asset helpers:

```
// app/javascript/my_pack/example.js.erb

<% helpers = ActionController::Base.helpers %>
var railsImagePath = "<%= helpers.image_path('rails.png') %>";
```

This is enabled by the `rails-erb-loader` loader rule in `config/webpack/shared.js`.

## Ready for React

To use Webpacker with React, just create a new app with `rails new myapp --webpack=react` (or run `rails webpacker:install:react` on a Rails app already setup with webpacker), and all the relevant dependencies
will be added via yarn and changes to the configuration files made. Now you can create JSX files and
have them properly compiled automatically.

## Ready for Angular with TypeScript

To use Webpacker with Angular, just create a new app with `rails new myapp --webpack=angular` (or run `rails webpacker:install:angular` on a Rails app already setup with webpacker). TypeScript support and the Angular core libraries will be added via yarn and changes to the configuration files made. An example component written in TypeScript is also added to your project in `app/javascript` so that you can experiment Angular right away.

## Ready for Vue

To use Webpacker with Vue, just create a new app with `rails new myapp --webpack=vue` (or run `rails webpacker:install:vue` on a Rails app already setup with webpacker). Vue and its supported libraries will be added via yarn and changes to the configuration files made. An example component is also added to your project in `app/javascript` so that you can experiment Vue right away.

## Troubleshooting

*  If you get this error `ENOENT: no such file or directory - node-sass` on Heroku
or elsewhere during `assets:precompile` or `bundle exec rails webpacker:compile`
then you would need to rebuild node-sass. It's a bit weird error,
basically, it can't find the `node-sass` binary.
An easy solution is to create a postinstall hook - `npm rebuild node-sass` in
`package.json` and that will ensure `node-sass` is rebuild whenever
you install any new modules.

* If you get this error `Can't find hello_react.js in manifest.json`
when loading a view in browser it's because Webpack is still compiling packs.
Webpacker uses a `manifest.json` file to keep track of packs in all environments,
however since this file is generated after packs are compiled by webpack. So,
if you load a view in browser whilst webpack is compiling you will get this error.
Therefore, make sure webpack
(i.e `./bin/webpack-watcher` or `./bin/webpack-dev-server`) is running and has
completed the compilation successfully before loading a view.

## Wishlist

- Improve process for linking to assets compiled by sprockets - shouldn't need to specify
` <% helpers = ActionController::Base.helpers %>` at the beginning of each file
- Consider chunking setup

## License
Webpacker is released under the [MIT License](https://opensource.org/licenses/MIT).
