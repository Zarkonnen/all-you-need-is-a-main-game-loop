# All you need is a main game loop

Want to try out game development? Have you tried out half a dozen tools and frameworks and engines, only to be overwhelmed each time?

I am going to strip it down: All you need is a main game loop.

Fundamentally, games work by keeping track of the state of the game world and then doing the following three things over and over again: processing user input, updating the game state, and displaying the game to the player.

The game state is all the information that has to be kept track of for the game. For example, with chess, this is the position of the pieces. With tic-tac-toe (noughts and crosses), it's a grid of nine fields that can contain an X, an O, or be blank. In both cases, the state also keeps track of whose turn it is. With a Flappy Bird game, the game state boils down to the position and velocity of the bird and the position of the pipes.

After each iteration of the main game loop, the game waits until it is time to go around again. It's fairly standard for a game to loop around sixty times per second. Depending on the game, nothing might happen in most of these frames - chess players usually take more than sixteen milliseconds to make a move. But that's fine. If nothing's changed, the game just re-displays the same game state and waits for the next cycle.

## Loop Phases

### Input
In the input phase, you check if the player is holding down any keys or clicking somewhere on the screen. On the basis of that, you change the game state. In chess, clicking on a piece (clicking on a part of the screen that a piece is displayed at) could select it, and clicking on an empty tile would execute a move. In Tic-Tac-Toe, clicking on one of the nine fields fills it with an X or O. In both cases, the game also changes which player's turn it is. With Flappy Bird, clicking on the screen causes the bird to accelerate upwards, that is, we add a predefined value to its y-velocity.

### Update
In the update phase, the game state advances by itself. With Chess and Tic Tac Toe, this just involves checking whether a player has won or the game is a draw. In Flappy Bird, the bird's position is updated on the basis of its velocity, and the game checks for collisions with pipes. This can be as simple as checking whether the bounding rectangles (the smallest rectangles still containing the whole thing) of the bird and a pipe overlap.

### Display
In the display phase, the game state is displayed to the user. Typically, this is done by drawing on the screen and playing sound effects, but it could also be done with raw text output, or audio cues, or, uh, an interpretative dance by a set of tiny robots. Anyway, drawing on the screen: at a basic level, a graphics system lets you draw colored shapes such as rectangles, circles and lines, as well as text and images. 3D engines are obviously a bit more complicated, but ultimately still come down to drawing colored polygons.

So in chess, displaying the game state means drawing the board, and then each piece in its place. If the board is 800x800 pixels, you start out by drawing a single white 800x800 square. Then, you draw 32 100x100 black squares to make a chessboard pattern. Next, you go over each tile, and if it contains a piece, draw it in the appropriate location.

At the start of the game, the black rook is in the top left of the board. This means a 100x100 picture of a black rook would be drawn at x = 0, y = 0. (Pixel coordinates are counted from the top left, with the x-axis going right and the y-axis going down.) The white king is at the bottom of the board, in the fourth rank. This means that we want to draw a picture of a white king at x = 300, y = 700. (Why 300 and not 400? Because the first piece in the row is drawn at x = 0, so the second's at 100, the third at 200, and the fourth at 300.)

## Examples
In this section, I'll go through some very simple games written in HTML5.

### Flappy Dot
[This](http://zarkonnen.github.io/all-you-need-is-a-main-game-loop/flappydot.html) is a simplified "flappy bird" game. There are no pipes: the only thing you have to do is keep the dot from hitting the top or bottom of the screen.

[Have a look at the code.](https://github.com/Zarkonnen/all-you-need-is-a-main-game-loop/blob/gh-pages/flappydot.html) For now, ignore everything outside the section labelled "THE GAME ITSELF". The other bits are boilerplate to set up a main game loop in HTML5.

#### Game State
First, there are three constants:

* gravity, which is expressed as pixels per second squared
* lift, which is the dot's upwards acceleration while X is pressed
* the size of the dot in pixels

Then, there are two variables, which are the whole game state:
* the vertical position of the dot in pixels from the top
* the vertical speed of the dot in pixels per second

#### Input
There is a single input rule, which is that if X is being pressed, the dot accelerates upwards. `lift` is multiplied with `msSinceLastUpdate` here, which is how many milliseconds have passed since we last went around the game loop.

#### Update
This does a bunch of things:

* Apply gravity, which works exactly the same as lift, but in the opposite direction.
* Update the dot's position based on its speed.
* Finally, the update function checks whether the dot has hit the top or the bottom of the screen. When that happens, the game gets reset.

#### Display
Here, we start out by drawing a blue background covering the whole of the game screen, then a white dot at the y-position given in the game state (and at a fixed horizontal position). Finally, we also draw some instruction text at x = 10, y = 20.

And that's the whole game - if you want to call it a game. But it has all the basics: it reacts to user input, changes over time, and draws to the screen. You could easily extend it by adding controls for horizontal movement, for example.
 
### Tic Tac Toe
The second example we'll go through is a more complete game: [tic-tac-toe](http://zarkonnen.github.io/all-you-need-is-a-main-game-loop/tictactoe.html) (noughts and crosses for the British reader). [Here's the code.](https://github.com/Zarkonnen/all-you-need-is-a-main-game-loop/blob/gh-pages/tictactoe.html)

#### State
Here, the state is a 3x3 grid of fields called `xo`, with each of the field cells initially set to `"-"`, meaning it hasn't been claimed by either player. In addition, there are three more pieces of information: the current player, the identity of the game's winner, if any, and whether the game has ended in a draw.

#### Input
Unlike as in the Flappy Dot example, this game uses mouse clicks for input. The game loop sets the `click` variable to the cursor x/y position if the user has clicked on the screen during this frame.

If the game's already ended due to there being a winner or a draw, the click is a signal to reset the game state and begin a new game. Otherwise, we want to figure out which cell the player clicked on. Each cell is displayed as a 100x100 square for a total 300x300 pixel display, so we can just divide the click position by 100 and round down to find the right cell.

The, if the cell is free, we can fill it with the player's symbol and switch to the other player.

#### Update
This function may look hairy, but it's actually very simple. I intentionally wrote it out in "long form" rather than using a more compact loop. What it boils down to is checking whether there are any horizontal, vertical or diagonal triplets of cells with the same symbol in - that isn't `-`! If so, that symbol - `X` or `O` - is the winner.

It also needs to check if the game is a draw, which it does simply by checking if all fields have been filled in.

#### Display
Finally, this is how the grid is drawn. It could be prettier, but this works for our purposes. We again start out by filling the entire game screen, 300x300 pixels in this case, with a single colour.

Next, we loop over the cells and draw the content of each cell, using a big 80px font.

Finally, we use a smaller font to draw a line of information at the top, announcing the winner, or a draw, or whose turn it is.

#### Putting it together
This is a complete, functioning tic-tac-toe game. It displays the grid, accepts user input to fill in cells, switches between players `X` and `O` after each move, and checks whether the game has been won or ended in a draw.

## Next steps
A chess game would be absolutely just more of the same! A bigger grid and more symbols (the pieces). A two-stage process to making a move: click on a piece to pick it up, and click on a legal tile to move it there.

There would be more complex logic about what moves are legal and what constitutes victory, of course, but that's it. (Don't believe me? When I have time I'll code it up right in the same HTML5 "framework".)

Speaking of this "framework" - [here](https://github.com/Zarkonnen/all-you-need-is-a-main-game-loop/blob/gh-pages/template.html) is a blank version of the code that contains no game at all, for you to build upon. Fork it and give it a try!
