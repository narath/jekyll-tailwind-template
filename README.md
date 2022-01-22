## Create a blank jekyll site

```
gem install bundler jekyll
jekyll new newsite --blank
cd newsite
rbenv local 3.1.0
```

Unfortunately jekyll does not yet play nice with Ruby 3.x, but is easily fixed by adding to the Gemfile

```
bundle init
bundle add jekyll webrick
bundle install
```

Let's add the binstubs so we can just type `jekyll s` instead of `bundle exec jekyll s`

```
bundle binstubs --all
rbenv rehash
```

Now you should be able to serve the test site
```
jekyll s
```

## Install Tailwindcss

These tutorials were really helpful but no longer completely up to date:

- https://stevenwestmoreland.com/2021/01/using-tailwind-css-with-jekyll.html
- https://mdoliwa.com/how-to-setup-jekyll-with-tailwind-css
- https://www.ocordova.me/blog/jekyll-with-tailwind/

Install the npm modules

```
npm install tailwindcss cssnano postcss postcss-import autoprefixer --save-dev
```

Use the [jekyll-postcss](https://github.com/mhanberg/jekyll-postcss) plugin.

Add this to your `Gemfile`:

```
group :jekyll_plugins do
  gem 'jekyll-postcss'
end

```

Add this to `_config.yml`:

```
plugins:
  - jekyll-postcss

postcss:
  cache: false
```

Note: I disable caching here because it looks like Tailwindcss now scans your files for classes that are needed and only includes those that are necessary. In order to have the jekyll reloading work in development I have disabled the caching of the css files.

Create the `postcss.config.js` file:

```js
const cssnano = require('cssnano')({ preset: 'default' })

module.exports = {
  plugins: [
    require('postcss-import'),
    require('tailwindcss'),
    require('autoprefixer'),
    ...(process.env.JEKYLL_ENV == "production" ? [cssnano] : [])
  ]
}
```

Create the tailwind config file

I use the initializer so I can tell if there are configuration changes in the latest version of tailwindcss.

```
npx tailwindcss init
```

I edited mine to look like this:

```js
module.exports = {
  content: [
    './_includes/**/*.{html,js,md}',
    './_layouts/**/*.{html,js,md}',
    './_posts/*.{html,js,md}',
    './*.{html,js,md}'
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

Note: most of the examples I saw listed `*.html` but since I mostly used `.md` I had to include this in the wildcard search so Tailwind would create the class for it in my css.

Update your `assets/css/main.scss` file:

```scss
---
---
@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";
```

Note: I could not get this to work with `_sass/main.scss`. I kept on getting an error stating that postcss-import could not find the file main. So I am just not using `_sass` folder (and I have deleted it).

The default layout includes the stylesheet reference:

```html
    <link rel="stylesheet" href="{{ "/assets/css/main.css" | relative_url }}">
```

So you can test your installation by adding some Tailwind css to your `index.md` file

```md
---
layout: default
title: "Happy Jekylling!"
---

<div class="flex h-screen">
  <div class="m-auto">
    <h1 class="text-5xl">I'm using Tailwind CSS!</h1>
  </div>
</div>
```

Now serve it locally

```
jekyll s
```

To prepare it for production don't forget to set the right environment variables for jekyll, its plugins and the node plugins.

```
JEKYLL_ENV=production NODE_ENV=production bundle exec jekyll build
```
