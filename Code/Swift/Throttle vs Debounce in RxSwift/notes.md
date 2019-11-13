  # [Throttle vs Debounce in RxSwift](https://medium.com/fantageek/throttle-vs-debounce-in-rxswift-86f8b303d5d4)

- Situation
    - There is a button. The user can click on it.
        - Throttle - specified to 2 seconds
            - User taps continually taps the button
            - Prints out "Tap" every 2 seconds
            - Basically calls the original function at most once per 2 seconds
        - Debounce - specified to 2 seconds
            - User taps continually taps the button and stops
            - After the user stops, "Tap" is printed out 2 seconds after
            - Basically calls the original function after the caller stops calling the action function after a specified period
