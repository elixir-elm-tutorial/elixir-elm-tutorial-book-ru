# Our First Game

Let's start creating our first minigame with Elm. We want to begin with
something small and simple that still has the characteristics and features we
want for our games.

Our initial game should be (very) small, self-contained, interactive, and fun.
And we'll also want to add a simple scoring mechanism so we can work towards
tracking player scores and sending that data to our back-end platform.

## Creating an Initial Game File

Inside the `lib/platform/web/elm` folder, let's create a new file named
`Game.elm`. We can initialize it with the following code so we'll have
something to display on the page:

```elm
module Game exposing (..)

import Html exposing (..)


main : Html msg
main =
    text "Elm Game"
```

## Creating a New Page for Games

On the Phoenix side, we'll need to create a new page where we can display our
game. We'll start by manually creating a new page and route, and later we'll
work towards a more flexible approach.

Inside the `lib/platform/web/templates/page` folder, let's create a new file
called `game.html.eex`. This will be very similar to what we did for our Elm
home page, and this time we'll just be creating a container div element for our
Elm game. This is all we need to add for the `game.html.eex` file:

```embedded_elixir
<div class="elm-game-container"></div>
```

Now let's update our `router.ex` file inside the `lib/platform/web` folder.
For now, we can add a simple route so that users can access our game by going
to `/elm/game` in the browser. Update the `router.ex` file with the following:

```elixir
scope "/", Platform.Web do
  pipe_through :browser

  get "/", PlayerController, :new
  get "/elm", PageController, :index
  get "/elm/game", PageController, :game
  resources "/players", PlayerController
  resources "/sessions", PlayerSessionController, only: [:new, :create, :delete]
end
```

When we visit `http://0.0.0.0:4000/elm/game` in our browser, we'll be able to
see the Elm game that we're creating. But we still need to take a couple
more steps before this works properly.

Let's update our `PageController` with a function that will render our new game
page. In the `lib/platform/web/controllers` folder, update the
`page_controller.ex` file with the following:

```elixir
defmodule Platform.Web.PageController do
  use Platform.Web, :controller

  plug :authenticate when action in [:index]

  def index(conn, _params) do
    render conn, "index.html"
  end

  def game(conn, _params) do
    render conn, "game.html"
  end

  defp authenticate(conn, _opts) do
    if conn.assigns.current_user() do
      conn
    else
      conn
      |> put_flash(:error, "You must be logged in to access that page.")
      |> redirect(to: player_path(conn, :new))
      |> halt()
    end
  end
end
```

## Configuring our Game Page

So we have our template, route, and controller configured properly, and we just
have a couple more small steps to take before we can see it rendered in the
browser.

First, we'll need to update our `brunch-config.js` file so that both `Main.elm`
and `Game.elm` will both be compiled (note that the only change here is on the
`elmBrunch` line).

```javascript
exports.config = {
  files: {
    javascripts: { joinTo: "js/app.js" },
    stylesheets: { joinTo: "css/app.css" },
    templates: { joinTo: "js/app.js" }
  },
  conventions: { assets: /^(static)/ },
  paths: {
    watched: ["../lib/platform/web/elm", "static", "css", "js", "vendor"],
    public: "../priv/static"
  },
  plugins: {
    babel: { ignore: [/vendor/] },
    elmBrunch: { elmFolder: "../lib/platform/web/elm", mainModules: ["Main.elm", "Game.elm"], outputFolder: "../../../../assets/vendor" }
  },
  modules: { autoRequire: { "js/app.js": ["js/app"] } },
  npm: { enabled: true }
};
```

Lastly, we can update our `app.js` file to render our game inside the Phoenix
application. At the bottom of our `assets/js/app.js` file, let's update our
code to look like this:

```javascript
const elmContainer = document.querySelector(".elm-container");
const elmGameContainer = document.querySelector(".elm-game-container");

if (elmContainer) {
  const elmApplication = Elm.Main.embed(elmContainer);
}

if (elmGameContainer) {
  const elmGame = Elm.Game.embed(elmGameContainer);
}
```

At this point, we should finally be able to see our new Elm game rendered in
the browser:

![Elm Game Page](images/our_first_game/elm_game_page.png)

You also might be thinking, "This is the worst game ever. It literally just
says 'Elm Game' and that's it." You're right, and in the next sections we're
going to set up our Elm application, add SVG for our game's background, add a
little character to work with, and wire up the keyboard for interaction.

## Base Application for Our Game

We've already got some experience in the preceding chapters with the Elm
Architecture, so we're not going to cover it in great detail here. Instead,
we're going to start by pasting in the following code, which will give us some
starter code to work with. Add the following to the `Game.elm` file:

```elm
module Game exposing (..)

import Html exposing (Html, div, text)


-- MAIN


main : Program Never Model Msg
main =
    Html.program
        { init = init
        , view = view
        , update = update
        , subscriptions = subscriptions
        }



-- MODEL


type alias Model =
    {}


initialModel : Model
initialModel =
    {}


init : ( Model, Cmd Msg )
init =
    ( initialModel, Cmd.none )



-- UPDATE


type Msg
    = NoOp


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        NoOp ->
            ( model, Cmd.none )



-- SUBSCRIPTIONS


subscriptions : Model -> Sub Msg
subscriptions model =
    Sub.none



-- VIEW


view : Model -> Html Msg
view model =
    div [] [ text "Elm Game Base" ]
```

There are different conventions one can use to set up their Elm applications,
but this is a minimal base application that I like to start with.

We have all the things we need to get started:

- minimal imports.
- a `main` function to wire everything together.
- an empty `Model` and `init` function.
- a default `update` function with a "NoOp" placeholder case that allows us
  to perform no operation. This may seem confusing at first, but it's helpful
  to have a default message to work with before we start updating our model.
- an intitial `subscriptions` function that starts with `Sub.none`.
- a simple `view` that contains a container `div` with some text that we can
  see to ensure that our application is working.

## Creating a Game Canvas

Before we can add our game character, we need to create a window where our hero
(or heroine) can live. We may decide on an alternative approach later, but for
now let's use SVG to create our game world.

In order to work with Elm's SVG library, we'll need to install the package and
import it into our project.

From the command-line, let's switch to the `lib/platform/web/elm` folder and
run the following command:

```shell
$ elm-package install elm-lang/svg
```

After agreeing to install the package by entering the `Y` key, here's the
output we should see:

```shell
$ elm elm-package install elm-lang/svg
To install elm-lang/svg I would like to add the following
dependency to elm-package.json:

    "elm-lang/svg": "2.0.0 <= v < 3.0.0"

May I add that to elm-package.json for you? [Y/n] Y

Some new packages are needed. Here is the upgrade plan.

  Install:
    elm-lang/svg 2.0.0

Do you approve of this plan? [Y/n] Y
Starting downloads...

  ● elm-lang/svg 2.0.0

Packages configured successfully!
```

Now that we have the package installed, let's import it at the top of our
`Game.elm` file. We'll import all of the `Svg` functions along with all the
`Svg.Attributes` functions since we'll be using quite a few of them. Update
the top of your `Game.elm` file with the following code:

```elm
module Game exposing (..)

import Html exposing (Html, div)
import Svg exposing (..)
import Svg.Attributes exposing (..)
```

## Setting Up a Game Window

Now we can create a small window to use for our game. We're going to create
a rectangle that has a 600px width and a 400px height. The rectangle will
reside our SVG element, and that will reside inside our HTML code.

You don't need to type this code in, but here's an HTML visualization that
might help if you're familiar with HTML structure:

```html
<html>
  <body>
    <div>
      <svg>
        <rect> <!-- We'll create our game inside this rectangle. -->
      </svg>
    </div>
  </body>
</html>
```

Even though we're using SVG for our game, we still want to nest it inside an
HTML document with a container `div` element. The reason for this is that we
might want to add HTML elements outside the game, or we could even add multiple
games to a single page if we wanted to.

Also, don't worry too much if you're unfamiliar with SVG. If you have
experience working with HTML elements and attributes and values then it's easy
to pick up. There are some quirks to working with SVG, but we'll learn what we
need to learn to start creating our Elm games so we can keep moving. But feel
free to take a look online because there are some great SVG learning materials
and courses available.

Let's add our SVG code to our Elm view, and we'll walk through how it all fits
together. At the bottom of the `Game.elm` file, add the following:

```elm
view : Model -> Html Msg
view model =
    div [] [ viewGame ]


viewGame : Svg Msg
viewGame =
    svg [ version "1.1", width "600", height "400" ] [ viewGameWindow ]


viewGameWindow : Svg Msg
viewGameWindow =
    rect
        [ width "600"
        , height "400"
        , fill "none"
        , stroke "black"
        ]
        []
```

We start with a `div` element at the top, which contains our `svg` element.
There is some duplication when we add our `width` and `height` attributes in
two separate places, but for now we'll just keep going so we can get something
rendered on the page. The `svg` element contains the `rect` element that will
serve as our small game window. Note that we need to add the same `width` and
`height` for both of these elements, or else we'd run the risk that the size
of our rectangle might exceed the size of our surrounding `svg` element.

Lastly, we add a `stroke` attribute to see a `"black"` line around our game window,
and a `fill` attribute with an initial value of `"none"`.

## Adding the Sky and the Ground

...