# Why Elmish needs Effects
*Subscriptions*.

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
The subscription function is called after init (as in Elmish) and _after each call to update_. The model is passed to the subscription function so that the Sub(s) returned can be customized depnding on the state of the model. The Sub returned (probably a batch created with Sub.batch) is compared to the previously returned Sub to determine which subscriptions are no longer needed, which are to remain and which are new. In other words, subscriptions are managed similarly to the DOM - only changes are reflected. As with the DOM, this is complely transparent to the program author.


