I recently started using the sway window manager. Although I've seen it pop up over the years, I decided to try it more seriously only recently, because after agreeing to an Ubuntu upgrade popup window, many things became broken in my install. Here are some notes and reflections from my experience going into sway.

Before this, I was using the default (gnome-shell) WM provided by Ubuntu 22.04.1 (Jammy). Before the upgrade, I was using 20.04 LTS with regolith, which is a customized variant of the i3 window manager. Before regolith, I mainly used xmonad.

in sway:

- centralized documentation is thinner than i3. Even though I found some descriptions of sway as a drop-in replacement of i3, coming from regolith linux made the configuration almost entirely invalid for sway, and it was much more efficient to do what the documentation suggests: make a copy of the sample config and go from there.
  - some online answers border on flippant RTFM hostility, which is a huge turn off
    - I suppose after sway becomes more widely adopted, to the level of i3, when people start showing off screenshots of their desktops, maybe it will feel more welcoming
  - this means I started with all default keybindings, and replaced them piecemeal with my preferred bindings
  - the positional bindings use VIM hjkl by default, which was nice default for me. IIRC, i3 uses jkl; by default, which I rebound to VIM.
- overall, the system feels snappier
- getting IME to work at all required me to actually learn some fcitx commands. I never had to learn how to interact with fcitx and I don't mind not knowing. But in this respect the user has to fill in many more blanks.
- multi monitor support, while harder to configure than when using X (I used to use arandr to get xrandr commands, and call them from a script, which was easy from a software standpoint, but the hidpi sometimes gets messed up, such that chromium/vscode/electron end up having 2x scaled windows, while firefox/emacs/thunar were scaled properly; I never figured out the cause; restarting the affected applications "fixed" the scaling issue), is far more reliable, and almost feels faster than other systems, like mac
  - moreover, being able to bind workspaces to specific monitors means the arrangement is applied immediately once the external monitor is detected. this is a small but wonderful quality of life improvement
  - possibly a bad side effect of how detection works, I encountered a very annoying issue when simultaneously attaching 1 monitor to 2 different systems (linux machine running sway + macbook pro). Once I switched the monitor's input to the mac's line, and switch back, sway seems to pick up a change in the monitor signal, and momentarily turns off the external monitor's output. The monitor then thinks there is no signal, and automatically switches back to the mac's input. The only way I was able to switch back to the linux machine with the external monitor on was to put the mac to sleep, switch to the linux input, which it now detects and displays correctly, then wake up the mac.
      - I attribute this to sway because it has not happened with i3 nor the ubuntu-native WM
- 

