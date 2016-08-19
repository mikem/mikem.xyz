---
layout: post
title: Passing Arguments to an Elm Application
---

You're starting to play with Elm and have come up with an app which you'd like to start using with a backend. One approach is to have the backend serve an HTML file which loads the JavaScript containing the compiled Elm code and initialize it with `Elm.Main.fullscreen()`:

```html
<!DOCTYPE HTML>
<html>
  <head>
    <title>My Elm App</title>
    <script src="app.js"></script>
  </head>

  <body>
    <script type="text/javascript">
      Elm.Main.fullscreen();
    </script>
  </body>
</html>
```

The app makes API calls to your backend to retrieve data to render, and you soon realize that it needs to hit `localhost` in development and `yoursite.com` in production. We can do this by passing data to the Elm application when it's iniaizlied. But how?

In [An Introduction to Elm](http://guide.elm-lang.org/) you'll notice `beginnerProgram` or `program` as the entry point in the examples. The type of these two functions is `Program Never`. A `Program` value "captures all the details needed to manage your application." `Program Never` essentially means this program takes no flags. We'll need to use `programWithFlags` instead:

```haskell
type alias Flags =
    { apiEndpoint : String }
    
    
main : Program Flags
main = 
    App.programWithFlags
        { {- assign init, view, update & subscriptions  -}
        }
```

What does the program do with the flags it receives? They're passed on to the `init` function. Change your `init` function to take flags as its argument:

```haskell
type alias Model =
    { apiEndpoint : String, ... }
    
    
init : Flags -> ( Model, Cmd Msg )
init flags =
    {- use flags, store the apiEndpoint on the model,
       or pass it to the function which retrieves
       data from your backend -}
```

That's all you need to change in your Elm app. All that remains is passing the `apiEndpoint` when the app is being initialized:

```html
    <script type="text/javascript">
      Elm.Main.fullscreen({apiEndpoint: "localhost:4567"});
    </script>
```

Of course you'd probably use a templating engine to pass the actual endpoint at runtime.

And there you go. As the next step, have your backend pass the data needed to render the initial page so the browser doesn't have to immediately make another round trip to the server for it.

# References

- [Documentation for the `Program` type](http://package.elm-lang.org/packages/elm-lang/core/4.0.5/Platform#Program)
- [Documentation for the `Never` type](http://package.elm-lang.org/packages/elm-lang/core/4.0.5/Basics#Never)
- [Documentation for the `programWithFlags` function](http://package.elm-lang.org/packages/elm-lang/html/1.1.0/Html-App#programWithFlags)
- [Mailing list thread with examples](https://groups.google.com/forum/#!topic/elm-discuss/E_2If1LI5OU)
