<!-- Copyright © SixtyFPS GmbH <info@slint.dev> ; SPDX-License-Identifier: MIT -->

# Creating The Tiles From JavaScript

This step places the game tiles randomly.

The code takes the list of tiles, duplicates it, and shuffles it, accessing the `memory_tiles` property through the JavaScript code.

As `memory_tiles` is an array, it's represented as a JavaScript [`Array`](https://slint.dev/docs/node/).
You can't change the model generated by Slint, but you can extract the tiles from it, and put them
in a [`slint.ArrayModel`](https://slint.dev/docs/node/classes/arraymodel.html) which implements the [`Model`](https://slint.dev/docs/node/interfaces/model.html) interface.
`ArrayModel` allows you to make changes and you can use it to replace the static generated model.

Change `main.js` to the following:

```js
{{#include main_tiles_from_js.js:main}}
```

Running this code opens a window that now shows a 4 by 4 grid of rectangles, which show or hide
the icons when a player clicks on them.

There's one last aspect missing now, the rules for the game.

<video autoplay loop muted playsinline src="https://slint.dev/blog/memory-game-tutorial/creating-the-tiles-from-rust.mp4"></video>
