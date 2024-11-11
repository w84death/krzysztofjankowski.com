Game of Life for BBC Micro:bit

This incredible little machine is both cute and intresting. What you can do with 16Mhz, 16KB RAM and a screen capable of 5x5 resolution?

![Game of Life on micro:bit](assets/microbit/gol1.gif)
![Game of Life on micro:bit](assets/microbit/gol2.gif)

## You can make Game of Life!

I don't have the real unit yet so I'm using web emulator. But to make it more challanging and fun I'm using only Microsoft Block Editor to program micro:bit controller.

![block-ide](assets/microbit/ide.png)

My first attemt was to use two-dimmensional arrays. But blocks don't support this well. Also it takes more ram. The solution was to use more calculations on a flat array. The screen resolution is 5x5 that gives us 25 cells. Then I created second array for sorrounding cells positions. This simplifies the code. Less code equals to less blocks in the editor. Final code turns out very nice. Both as algorithm and visual structure. And blinking LED are always cool.

[![source-code](assets/microbit/code776.png)](assets/microbit/code.png)

## Radio

Another cool thing about micro:bit and it's software is radio module. It is the simplies way to communicate between multiple devices I have ever seen and used. It's super fun. I tested it implementing sharing feature for this Game of Life. Each moment you can pause and share world state with another micro:bits.

## Not Only For Kids

BBC Micro:bit is designed for kids. But is as usefull for adults as well. I woudl never made this kind of solution for Game of Life before using e.g. Python. This turns out as a nice, refreshing exercise/experiment. And as a bonus when I finally buy real device I will have a nice code to run.


## Play

- [Shake] to generate new random state.
- [A] send state to other devices via radio
- [B] start/stop simulation
- [A]+[B] print generation number

<div style="position:relative;height:0;padding-bottom:81.97%;overflow:hidden;"><iframe style="position:absolute;top:0;left:0;width:100%;height:100%;" src="https://makecode.microbit.org/---run?id=_YiR8bMifY5y9" allowfullscreen="allowfullscreen" sandbox="allow-popups allow-forms allow-scripts allow-same-origin" frameborder="0"></iframe></div>

Tags: microbit, code, Microsoft-Block-Editor
