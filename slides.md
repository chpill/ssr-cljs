<!--- .slide data-background="images/clojure-logo.png" -->

# Server-Side Rendering
<!-- <img src="images/clojure-logo.png" class="no-style"/> -->

-----

## Who am I

**Etienne Spillemaeker** - Specimen from the JS planet

[github.com/chpill](https://github.com/chpill)

[twitter.com/chpill_](https://twitter.com/chpill_)


-----

### Some context: Single page application (SPA)

Offers more interactivity than static pages

Dynamically fetch and display content

Manipulates its own **routes/URLs**

Note:
The back button works in good SPAs!

-----

### The tradeoffs

SEO is really tricky for dynamic content

First rendering is slow:  
download JS -> interpret JS (-> fetch data) -> render

-----

### Even worst for Clojurescript

Generated JS payloads get big real fast (even with advanced optimizations and dead code elimination).

<svg style="vertical-align: middle" height="60" version="1.1" viewBox="0 0 12 16" width="45"><path fill-rule="evenodd" d="M4 9H3V8h1v1zm0-3H3v1h1V6zm0-2H3v1h1V4zm0-2H3v1h1V2zm8-1v12c0 .55-.45 1-1 1H6v2l-1.5-1.5L3 16v-2H1c-.55 0-1-.45-1-1V1c0-.55.45-1 1-1h10c.55 0 1 .45 1 1zm-1 10H1v2h2v-1h3v1h5v-2zm0-10H2v9h9V1z"></path></svg>
[chpill/cljs-weight](https://github.com/chpill/cljs-weight)


-----

|                          | JS   | GZIP |
|--------------------------|------|------|
| (js/alert "hello world") | 5K   | 2K   |
| 1 core.async channel     | 105K | 24K  |
| 1 clojure.spec spec      | 116K | 26K  |
| client routing           | 157K | 37K  |
| 1 Rum view               | 270K | 72K  |
| core async + websockets  | 480K | 116K |
|--------------------------|------|------|
| all together             | 680K | 175K |


-----

### Extreme case: web front-end of CircleCI

Project stats:  
26317 lines of cljs  
277 lines of clj (macros)

Weighs a staggering **3.4M** of JS (**1.02M** when gzipped)

![circleci production js request timing](images/circleci-js-payload-firefox-devtools-network-view-fs8.png)

-----

## What can we do?

-----

## Progressive web apps?

Targets a much harder problem: the mobile  
-> brings a lot of complexity

Does nothing for the very first render

Note:
PWA are a way to get some JS code to run "outside" of the context of a web page,
that can intercept the page requests. It enables you to do caching etc...

-----

## Server-side rendering (with react)
### A bit of history
<!-- <h3 class="fragment">A bit of history</h1> -->

-----

### The react router mega demo!
<svg style="vertical-align: middle" height="60" version="1.1" viewBox="0 0 12 16" width="45"><path fill-rule="evenodd" d="M4 9H3V8h1v1zm0-3H3v1h1V6zm0-2H3v1h1V4zm0-2H3v1h1V2zm8-1v12c0 .55-.45 1-1 1H6v2l-1.5-1.5L3 16v-2H1c-.55 0-1-.45-1-1V1c0-.55.45-1 1-1h10c.55 0 1 .45 1 1zm-1 10H1v2h2v-1h3v1h5v-2zm0-10H2v9h9V1z"></path></svg>
[ryanflorence/react-router-mega-demo](https://github.com/ryanflorence/react-router-mega-demo/)

<svg height=60 viewBox="0 25 310 260" style="enable-background:new 0 0 310 310; vertical-align:middle;" xml:space="preserve"> <path d="M297.917,64.645c-11.19-13.302-31.85-18.728-71.306-18.728H83.386c-40.359,0-61.369,5.776-72.517,19.938   C0,79.663,0,100.008,0,128.166v53.669c0,54.551,12.896,82.248,83.386,82.248h143.226c34.216,0,53.176-4.788,65.442-16.527   C304.633,235.518,310,215.863,310,181.835v-53.669C310,98.471,309.159,78.006,297.917,64.645z M199.021,162.41l-65.038,33.991   c-1.454,0.76-3.044,1.137-4.632,1.137c-1.798,0-3.592-0.484-5.181-1.446c-2.992-1.813-4.819-5.056-4.819-8.554v-67.764   c0-3.492,1.822-6.732,4.808-8.546c2.987-1.814,6.702-1.938,9.801-0.328l65.038,33.772c3.309,1.718,5.387,5.134,5.392,8.861   C204.394,157.263,202.325,160.684,199.021,162.41z" style="fill: rgb(0, 0, 0);"></path> </svg>
[the talk](https://www.youtube.com/watch?v=XZfvW1a8Xac)

Just run your react app on a nodeJS server, easy!
A web-app with links that work even without JS!

Note:
* First react conf, 30 january 2015
* show the demo locally

-----

### In the clojure world

The first attempts were... laborious

<svg style="vertical-align: middle" height="60" version="1.1" viewBox="0 0 12 16" width="45"><path fill-rule="evenodd" d="M4 9H3V8h1v1zm0-3H3v1h1V6zm0-2H3v1h1V4zm0-2H3v1h1V2zm8-1v12c0 .55-.45 1-1 1H6v2l-1.5-1.5L3 16v-2H1c-.55 0-1-.45-1-1V1c0-.55.45-1 1-1h10c.55 0 1 .45 1 1zm-1 10H1v2h2v-1h3v1h5v-2zm0-10H2v9h9V1z"></path></svg>
[pupeno/prerenderer](https://github.com/pupeno/prerenderer)

spawns a nodeJS process that runs the SPA

arbitrarly decide when the page is "rendered enough" (default: 300ms)

Note:
- Apparently, nashorn is really bad, especially at startup
- raises complexity of operations
- Some rails examples were better, with some pool of workers etc

-----

### Then one day at the conj 2015...

<img width=300 alt="arohner avatar" src="images/arohner-avatar.jpeg">

<svg style="vertical-align: middle" height="60" version="1.1" viewBox="0 0 12 16" width="45"><path fill-rule="evenodd" d="M4 9H3V8h1v1zm0-3H3v1h1V6zm0-2H3v1h1V4zm0-2H3v1h1V2zm8-1v12c0 .55-.45 1-1 1H6v2l-1.5-1.5L3 16v-2H1c-.55 0-1-.45-1-1V1c0-.55.45-1 1-1h10c.55 0 1 .45 1 
1zm-1 10H1v2h2v-1h3v1h5v-2zm0-10H2v9h9V1z"></path></svg>
[arohner/foam](https://github.com/arohner/foam)

Write your UI in cljc!

-----

### In clojure today

<svg style="vertical-align: middle" height="60" version="1.1" viewBox="0 0 12 16" width="45"><path fill-rule="evenodd" d="M4 9H3V8h1v1zm0-3H3v1h1V6zm0-2H3v1h1V4zm0-2H3v1h1V2zm8-1v12c0 .55-.45 1-1 1H6v2l-1.5-1.5L3 16v-2H1c-.55 0-1-.45-1-1V1c0-.55.45-1 1-1h10c.55 0 1 .45 1 
1zm-1 10H1v2h2v-1h3v1h5v-2zm0-10H2v9h9V1z"></path></svg>
[tonsky/rum](https://github.com/tonksy/rum)

<svg style="vertical-align: middle" height="60" version="1.1" viewBox="0 0 12 16" width="45"><path fill-rule="evenodd" d="M4 9H3V8h1v1zm0-3H3v1h1V6zm0-2H3v1h1V4zm0-2H3v1h1V2zm8-1v12c0 .55-.45 1-1 1H6v2l-1.5-1.5L3 16v-2H1c-.55 0-1-.45-1-1V1c0-.55.45-1 1-1h10c.55 0 1 .45 1 
1zm-1 10H1v2h2v-1h3v1h5v-2zm0-10H2v9h9V1z"></path></svg>
[omcljs/om-next](https://github.com/omcljs/om)


blog post: [rendering reagent on the server using hiccup](https://yogthos.net/posts/2015-11-24-Serverside-Reagent.html)
(if you are using re-frame, tough luck...)

Note:
re-frame uses a singleton `db` atom. Maybe something can be tried with dynamic
bindings but it would be seriously nasty.

-----

### This works on both the server and the client!

```
;; my-app.views.example.cljc
(ns my-app.views.example
 (:require [rum.core :as rum]))

(rum/defc header [name user-on-twitter]
  [:header
   [:p (str "Hello, I am " name)]
   [:p [:a {:href user-on-twitter} "Follow me on twitter!"]]])
```


-----

TODO schema of the code organisation

Note:
* Views and client-side routing must be in cljc files
* Clj side: HTTP server + whatever you need
* Cljs side: client "bootstrap" (mounting react components in the DOM, parsing the data)


-----

```
<html>
  <head>...</head>
  <body>
    <div id="my-app-container">
      {{{HTML of the app}}}
    </div>

    <script id="my-app-data" type="application/json">
      {{{data used to render HTML above, as json}}}
    </script>

    <script type="text/javascript" src="/js/my-app.js"></script>
  </body>
</html>

```

-----

### Pitfall #1

Be mindful of how you inject the data in the html page

```
<!-- If you do that, it will work at first... -->
<script type="text/javascript" >
  window.bootstrapData = {{{data used to render HTML above}}};
</script>

```

-----

### One day in production...

![Un beau jour en production](images/shining-JS-fs8.png)

-----

![Le coupable](images/shining-u2028-fs8.png)

-----

### JS and JSON: not the same character set...

For example `\u2028` (line separator) and `\u2029` (paragraph separator) are
legal in json, but illegal in a js file or script tag...

-> Put the json data in a script tag with a type "application/json"
```
<script type="application/json">
  {{{your app data as hson}}}
</script>
```

Note:

-----

### Pitfall #2

Make sure you use the same character encoding in the html and on your server!

```
<head>
  <meta charset="utf-8">
  ...
</head>
```

```
(-> {:body (server-side-render some-data)}
    (response/content-type "text/html")
    (response/charset "UTF-8")
```

-----

### How to wrap a react lib to get it working on the server

Example: wrapping React FlipMove
(Assuming you already have your extern file)

-----

First make it work without server-rendering

```
(ns my-app.views.flip-move
  (:require [rum.core :as rum]
             ;; This must match your extern declaration
            #?(:cljs com.react-flip-move.FlipMove))) 

(rum/defc wrapper [children]
  #?(:cljs (js/React.createElement js/FlipMove
                                   children)))
```

-----

Add some dummy code for the server

```
(ns my-app.views.flip-move
  (:require [rum.core :as rum]
             ;; This must match your extern declaration
            #?(:cljs com.react-flip-move.FlipMove))) 

(rum/defc wrapper [children]
  #?(:clj [:div children]
     :cljs (js/React.createElement js/FlipMove
                                   children)))
```

-----

React will yell at you because the server markup does not match its own

![react warning](images/react-warning.png)

-----

```
(ns my-app.views.flip-move
  (:require [rum.core :as rum]
             ;; This must match your extern declaration
            #?(:cljs com.react-flip-move.FlipMove))) 

(rum/defc wrapper [children]
  #?(:clj [:div 
           {:style {:position "relative"}}
           children]
     :cljs (js/React.createElement js/FlipMove
                                   children)))

```
