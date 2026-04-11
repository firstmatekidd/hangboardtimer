## TODO
* Should be responsive at 320px width so that it doesn't scroll left/right
* Ad banner at the bottom is not adhering to the fixed positioning requirement
* The timer never goes to 0 and the ring doesn't completely drain. Handle the edge case so that it goes completely to 0 and then resets to the next state, even if that means we don't see the ring completely full.
* Add a fanfare particle explosion when the done state is reached.


## DONE
* Use the wakeLock API to keep the screen from going to sleep while the app is in any states that are counting down.
* Choose a different green color so that it has proper color contrast with the white text
* Reset button should be higher contrast against the background and it's text.
* Settings should be editable while paused
* State display should have a bigger font size
* Hang count should have a bigger font size, especially in landscape.
* Increase color contrast in the resting and done states
* The start button lowers opacity on hover but the reset and help buttons brighten. Let's keep consistent and change the start button to also brighten
* implement haptics
* Fonts used have serifs and should be san-serif
* Change the countdown timer numbering to use the same font as the rest of the app
* No GET_READY state
* Landscape implemented
* Improve the visibility of the paused state. When paused, reduce the opacity of the countdown timer area.
* Start the ring full and display whatever the current hang time is set to instead of "--" on initial start
* Increase font size for state label, start and reset button text