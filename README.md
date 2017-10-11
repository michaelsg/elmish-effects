# Why Elmish needs Effects
Subscriptions *(and Commands to a lesser extent)*.

# Example
Take the [example of a clock for react-native](https://github.com/fable-elmish/sample-react-timer-svg/blob/master/src/App.fs#L25). The program is established with a subscription function which will be called once on program startup - right after init. 
```fsharp
let timerTick dispatch =
    window.setInterval(fun _ -> 
        dispatch (Tick DateTime.Now)
    , 1000) |> ignore


let subscription _ = Cmd.ofSub timerTick
```
(I have issues with ``Cmd.ofSub``, but I'll address that later - this does not exist in Elm, but that's not why I object)

What if this clock were not the entire application? What if only one page in your SPA needed a timer? How would you cancel the timer when it's no longer needed?

# How does Elm deal with Subscriptions?
The subscription function is called after init (as in Elmish) and _after each call to update_. The model is passed to the subscription function so that the Sub(s) returned can be customized depending on the state of the model. The Sub returned (probably a batch created with Sub.batch) is compared to the previously returned Sub to determine which subscriptions are no longer needed, which are to remain and which are new. In other words, subscriptions are managed similarly to the DOM - only changes are reflected. As with the DOM, this is complely transparent to the program author.

# Elm also uses Effects Managers for Cmds. Doesn't Elmish need that, too?
Yes and No. Elm Cmd Effects seem to have a lot to do with asynchrony. In Elmish ``Cmd.ofPromise`` is easily used to deal with the one-off Promise result. Elm, with its hostile view of all things non-Elm, does not allow such direct interaction. Still, using Effects Managers for Cmds adds one interesting ability - Cmd batching. As with subscriptions, Cmds are also handled in batches: after all Cmds are returned from update, they are passed in bulk to the Effects Mangers. An Effect Manager could choose to optimize the Cmds in some way.

# This is important to maintaining the Elm-like model of program design
(Add more here to convince)

# Bring on the Magic
Effects modules are treated specially in Elm. This is the one instance *I* know of where compiler magic is involved. Effects are grouped by Type. Each effects module declares its Type in the module header. All Effects (Subs and Cmds) are divided by Type and dispatched in bulk to the appropriate Effects Manager. The magic comes in how these Effect Types are hidden from the user. Even though `Time.every` creates a message of type Time.MyType the magic subscription function available to effects modules converts the private Effect Type into a Sub of the expected message type. By doing this, the user does not need to wrap each Effect Type in a private message. 

Effect Managers are run like small independent program loops. They each have a message queue dealing with their own message type (never exposed to a user program). The runtime maintains a state for each effect manager, passing the state into the 2 callback functions `onEffects` and `onSelfMsg`. Different and telling, these 2 functions return an asynchronously updated state. An init method is also exposed from each effect manager to create its initial state.

There are 2 more functions the compiler automatically uses in effects mangers: `subMap` and `cmdMap`. Calling Cmd.map or Sub.map tranparently calls the correct implementation based on the effect manger that created the Cmd or Sub.


