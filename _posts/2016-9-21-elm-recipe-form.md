---
layout: post
title: A Recipe Form in Elm
comments: true
---

I remember looking at Elm a while ago, but just never did anything with it. Then about 2 months ago, I joined a slack channel focused on the language, checked out a meetup, and just got really into it.  If you are interested in trying out a new language, I would highly recommend it. The community so far has been great and very enthusiastic, and the language itself has truly been a joy to learn. The [documentation](https://guide.elm-lang.org/) is very thorough and provided all of the information I needed to work on my first Elm project!

My first Rails crash course was a recipe app. I love to cook and always trying to make it easier for me to write and keep different recipes for dishes I make. I was never truly happy with the forms I was making using HTML and JQuery and have always thought of different ways to make a form for building and submitting a recipe. I decided to have this form be my Elm crash course.

I used the basic Elm structure `Model`, `Update` and `View` to set up my app. Beginning with the `Model`, I set up different states to keep track of:

```elm
type alias Model =
    { title : String
    , ingredient : String
    , instruction : String
    , ingredients : List String
    , instructions : List String
    , recipes : List Recipe
    , currentRecipe : Maybe Recipe
    , uid : Int
    }
```

In this model, `title`, `ingredient`, `instruction`, `ingredients` and `instructions` are needed to keep track of the state of the form. `recipes`, `currentRecipe`, and `uid` are to help keep track of an individual `Recipe` whose model I also had to create:

```elm
type alias Recipe =
    { id : Int
    , title : String
    , ingredients : List String
    , instructions : List String
    }
```

What I really liked about elm is creating types and defining what types they contain. This way if I create a function or a view requiring a different type, I will be notified and can easily debug from there. 

Next in `Update`, I created a series of message of the type `Msg` which pass functions impacting the `Model`. Basically if `Model` tracks state, `Update` is what changes it. Here are the functions being used in my app: 

```elm
type Msg
    = NoOp
    | Title String
    | Ingredient String
    | Instruction String
    | AddIngredient String
    | AddInstruction String
    | RemoveIngredient String
    | RemoveInstruction String
    | AddRecipe
    | GetRecipe Int
    
update : Msg -> Model -> Model
update msg model =
    case msg of
        NoOp ->
            model

        Title title ->
            { model | title = title }

        Ingredient ingredient ->
            { model | ingredient = ingredient }

        Instruction instruction ->
            { model | instruction = instruction }

        AddIngredient ingredient ->
            if isEmpty ingredient == True then
                model
            else
                { model
                    | ingredients = model.ingredients ++ [ ingredient ]
                    , ingredient = ""
                }

        AddInstruction instruction ->
            if isEmpty instruction == True then
                model
            else
                { model
                    | instructions = model.instructions ++ [ instruction ]
                    , instruction = ""
                }

        RemoveIngredient ingredient ->
            { model
                | ingredients =
                    List.filter (\n -> (n /= ingredient)) model.ingredients
            }

        RemoveInstruction instruction ->
            { model
                | instructions =
                    List.filter (\n -> (n /= instruction)) model.instructions
            }

        AddRecipe ->
            { model
                | recipes = model.recipes ++ [ newRecipe model.uid model.title model.ingredients model.instructions ]
                , title = ""
                , ingredients = []
                , instructions = []
                , uid = model.uid + 1
            }

        GetRecipe id ->
            { model | currentRecipe = List.head (List.filter (\n -> (n.id == id)) model.recipes) }
```

Notice how each `Update` function impacts a different part of the `Model`. These changes are called from and interact with elements in the `View`. Notice that some functions take a parameter which would correspond to input given in the form. Another thing I enjoyed about Elm: given that the model types and update functions are so clearly defined, I could look at these functions and get a general idea of what is being changed.

Finally to the `View` which provides input to `Update` and displays what is set in the `Model`:

```elm
view : Model -> Html Msg
view model =
    div []
        [ input
            [ type' "text"
            , class "title"
            , placeholder "Title"
            , value model.title
            , onInput Title
            ]
            []
        , div []
            [ h2 [] [ text "Ingredients" ]
            , input
                [ type' "text"
                , class "item-input"
                , placeholder "10 Granny Smith Apples..."
                , value model.ingredient
                , onInput Ingredient
                , onEnter (AddIngredient model.ingredient)
                ]
                []
            , listIngredients model
            , h2 [] [ text "Instructions" ]
            , input
                [ type' "text"
                , class "item-input"
                , placeholder "Peel the apples..."
                , value model.instruction
                , onInput Instruction
                , onEnter (AddInstruction model.instruction)
                ]
                []
            , listInstructions model
            , p [] <|
                recipeFilter
                    [ (,) (model.title /= "") <| button [ class "add-button", onClick (AddRecipe) ] [ text "Add Recipe" ]
                    ]
            ]
        , listRecipes model
        , showRecipe model.currentRecipe
        ]
```

As you can see, different parts of the model are being displayed and used here. The `onInput` and `onClick` functions are a part of the Elm core language and are examples of triggers that make changes defined in `Update`. functions such as `listIngredients` and `listInstructions` are `View` functions that allow for displaying of more complex elements. In this case it is to allow the list of ingredients and instructions to be parsed from the model and displayed individually. Here is an example of what one of them is doing:

```elm
listIngredients : Model -> Html Msg
listIngredients model =
    let
        parseIngredient : String -> Html Msg
        parseIngredient ingredient =
            li []
                [ text ingredient
                , button
                    [ class "remove-button"
                    , onClick (RemoveIngredient ingredient)
                    ]
                    [ text "Remove" ]
                ]
    in
        ul [] (List.map parseIngredient model.ingredients)
```

In this function, I am defining that a `Model` type needs to be returned as a `Html Msg` for display in the `View`.  Working from bottom up, I am running a `map` through the `model.ingredients` within a `ul` element.  I am using each individual `ingredient` located within that list as an argument for `parseIngredient`. This nested function takes `ingredient` and creates a `li` element for each one along with a remove button I created in case I want to get rid of it.

This is only part of what I created using Elm, I wanted to give an idea of the basic architecture and what it took to create this form.  The rest of the source can be viewed on my [GitHub Repo](https://github.com/mostlybadfly/recipe-form). You can also play around with the recipe form yourself: [recipe-form](http://www.mostlybadfly.com/recipe-form/)
