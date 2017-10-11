# Why Elmish needs Effects
*Subscriptions*.

# Example
Take the simple example of a clock for react-native (https://github.com/fable-elmish/sample-react-timer-svg/blob/master/src/App.fs#L25). The program is established with a subscription function which will be called once on program startup - right after init. 
```fsharp
let timerTick dispatch =
    window.setInterval(fun _ -> 
        dispatch (Tick DateTime.Now)
    , 1000) |> ignore


let subscription _ = Cmd.ofSub timerTick
```
(I have issues with ``Cmd.ofSub``, but I'll address that later - this does not exist in Elm, but that's not why I object)

What if this clock were not the entire application? What if only one page in your SPA needed a timer? How would you cancel the timer when it's no longer needed? You don't.

# How does Elm deal with Subscriptions?
The subscription function is called once after init (in in Elmish) and once after each call to update. The Sub returned (probably a batch created with Sub.batch) is compared to the previously returned Sub to determine which subscriptions are no longer needed, which are to remain and which are new.


