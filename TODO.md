## TODO
* Landscape not implemented
* Should be responsive at 320px width so that it doesn't scroll left/right
* Ad banner at the bottom is not adhering to absolute positioning requirement
* State display should have a bigger font size
* No GET_READY state
* Settings should be editable while paused
* Reset button should be higher contrast against the background and it's text.
* Increase font size for state label, start and reset button text
* Increase contrast ration between the text on the Start button and the start button's background color
* The start button lowers opacity on hover but the reset and help buttons brighten. Let's keep consistent and change the start button to also brighten
* The timer never goes to 0 and the ring doesn't completely drain. Handle the edge case so that it goes completely to 0 and then resets to the next state, even if that means we don't see the ring completely full.
* Add a fanfare particle explosion when the done state is reached.
* Choose a different green color so that it has proper color contrast with the white text
* Start the ring full and display whatever the current hang time is set to instead of "--" on initial start

## DONE
* implement haptics
* Fonts used have serifs and should be san-serif
* Change the countdown timer numbering to use the same font as the rest of the app