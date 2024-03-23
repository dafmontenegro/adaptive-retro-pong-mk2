# adaptive-retro-pong-mk2
The project consists of a new version of the classic game PONG released by Atari in 1972. This reinterpretation of the game makes use of p5.js (a JavaScript library for creative programming[1]), and proposes the implementation of adaptive difficulty that aims to always provide a challenge for the player. This is achieved by adjusting the game's speed, bar width, and precision of the automatic mode, all according to the player's performance during the game.

> This new version takes inspiration from the initial version of the project and aims to achieve improvements in the algorithm responsible for capturing the ball. 
If you want to see the first version of the project, please go to: [**adaptive-retro-pong**](https://github.com/dafmontenegro/adaptive-retro-pong)

## How to Play?

[**p5.js Demo:**](https://editor.p5js.org/dafmontenegro/full/Cnx7DpEdd) Play and edit the code in real-time from a p5.js project.

[**Demo:**](https://dafmontenegro.com/adaptive-retro-pong/) Play fullscreen without interruptions.

## Controls
When starting the game, you'll appear in the pause menu and have the following options to configure your game:

- **Mouse:**
  - **Cursor Movement:** Controls the bar(s) vertically in manual mode (manual mode is indicated by a dot in the center of the bar and the information on the screen `player#Auto: deactivated`).

- **Keys:**
  - **ENTER:** Pause/Resume the game.
  - **i:** Activate/Deactivate game information (`player1Auto`, `player2Auto`, `Speed`, `barHeight`).
  - **1 (Left Bar):** Activate/Deactivate automatic mode; when not in automatic mode, switches to manual, which means moving it with the mouse.
  - **2 (Right Bar):** Activate/Deactivate automatic mode; when not in automatic mode, switches to manual, which means moving it with the mouse.
  - **a:** Activate/Deactivate game audio.

> **Note:** The application is designed to work in landscape format and requires a keyboard (or input of characters) for proper configuration and functionality.

## Interface

- **Scoreboard:** Counters at the top half of each player, indicating each player's score. Each time a player misses a ball, one point is added to the other player's score.

## Interface + Information

- **player#Auto:** Indicates the current playing mode for player#, which can be automatic or manual, explained as follows:
    - **`0.0# - #.##` (automatic mode):** Defines the range of precision with which the player will automatically move each frame to intercept the ball relative to the center of the bar. <br>**Example:** Hypothetically, the player needs to move 100 pixels to align the center of the bar with the center of the ball during a frame. If the defined range is 0.03 - 0.30, then a random number between these two values will be generated, meaning that for that frame, the bar will move between 3 and 30 pixels.
    - **`deactivated` (manual mode):** Implies that the player is controlled with the mouse.

- **Speed:** Refers to how long the ball takes in seconds to move from one side of the screen to the other (cross the width of the screen or `windowWidth`).

- **barHeight:** Is the number determining the size of the bar relative to the game window. `(player#.height / windowHeight)` calculates the proportion of the player's bar height relative to the game window's height. <br>**Example:** A proportion of 0.16 implies that the bar has a height of 16% of the window's height.

> Bar heights, ball speed, and automatic precision ranges can vary during the game based on various factors such as scoring, player type (automatic or not), and implemented adaptive difficulty functions (the better you play, the harder the game will be and vice versa).

## Implementation Details

### Our Target Frame Rate

We have explicitly set the FPS target to 60 frames per second using the `frameRate(60)` function in the `setup()` function. This means that the animation should aim to run at a speed of 60 FPS.

### How is Responsiveness Achieved in the Application?

1. **Dynamic Canvas Size:** By creating the canvas with `createCanvas(windowWidth, windowHeight)`, the canvas automatically adjusts to the size of the browser window. This ensures that the game adapts to different screen sizes.

2. **Relative Dimensions and Positions:** The dimensions and positions of game elements are defined based on the window size, using values like `windowWidth`, `windowHeight`, and proportions of these values. For example, the ball size (`ballSize`) is defined as a percentage (`windowHeight * 0.02`) of the window's height, and the positions of players (`player1` and `player2`) are also defined based on the window size.

3. **Handling Screen Orientation:** The code checks the screen orientation with the condition `if (windowHeight < windowWidth)`. If the window's height is less than its width, it is assumed that the screen is in landscape mode, and the game is executed. If the height is greater than the width, a message is displayed indicating to the user to use the game in landscape mode.

### What Classes Were Used and Created in the Game?

1. **Ball (`class Ball`):** This class defines the behavior and properties of the ball in the game. It contains methods to initialize the ball's position and velocity, as well as to update its position in each frame (`update`). It also includes a method `show` to display the ball on the game canvas. Additionally, it has an `applyScore` method to handle scoring and adjust the speed and other properties of the game when a point is scored.

2. **Player (`class Player`):** This class defines the behavior and properties of the players in the game, i.e., the bars that users can control or that can be controlled automatically (`auto = true`). It contains methods to initialize the player's position, height, and precision, as well as to update its position in each frame (`update`). It also includes a `show` method to display the player on the game canvas. The player's precision (`accuracy`) and height (`height`) can be dynamically adjusted during the game.

### How is automatic mode able to capture the ball?

In the **Player.update()** method, the automatic player calculates its new position based on the `y-coordinate` of the ball as the direction of the ball. During a frame, it checks if the current trajectory will impact its side of the screen. If so, it adjusts its position according to `accuracy` as explained below; otherwise, it remains still.

This is the line of code responsible for making the mentioned adjustment:
```javascript
this.y += (ball.y - this.y) * random(this.accuracy/10, this.accuracy);
```
The idea is to calculate the distance at which the ball (`ball.y`) is from the center of the bar (`this.y`). This is done by `(ball.y - this.y)`, which can result in both positive and negative values, hence the use of `+=` in the formula. Once it's known how far the bar would need to move in `y` to align perfectly with the ball, a random number is generated within a defined range according to `this.accuracy`. This range is of the form `0.0# - #.##`, where `0.0#` is one-tenth of `this.accuracy` and serves as the lower limit, and vice versa, `this.accuracy` serves as the upper limit.

### When is the difficulty adjustment made?

The difficulty adjustment is performed in the `Ball.applyScore()` method. This method is called when a player scores a point and is responsible for updating various aspects of the game that contribute to the difficulty. The difficulty adjustments are as follows:

- **Precision of Automatic Players (`accuracy`):** In the `Ball.applyScore()` function, the precision of automatic players is adjusted according to the performance of the other player and whether they are automatic or not. A minimum and maximum range for precision (`minAccuracy` and `maxAccuracy`) is defined, and it is dynamically modified during the game, making them more or less effective in intercepting the ball.

- **Ball Speed (`speed`):** The ball speed is initialized with a value of 3 seconds, referring to the time it takes for the ball to move from one side of the screen to the other. It is adjusted according to the players' performance, affecting the perceived speed at which the ball moves across the screen.

- **Bar Heights (`height`):** The heights of the players' bars are initially set relative to the window's height (`windowHeight`). In the `Ball.applyScore()` function, it is dynamically adjusted in response to the players' performance, maintaining a relative balance between them.

> Depending on whether the opposing player is controlled automatically or by a human player, various game parameters are adjusted to increase or decrease difficulty.

## Contributions
Contributions are welcome! If you have ideas to improve the game or find any errors in our code, feel free to reach out and share your feedback.

## References
[1] p5.js. Retrieved from: [https://p5js.org/](https://p5js.org/)
