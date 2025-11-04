# Endless Road to the Hotel: Custom Side-Scroller Game \- Design Document

## **1\. Overview**

This document outlines the design, architecture, and technology for a "Chrome Dinosaur" style side-scrolling runner game. The primary goals are to create a simple, stateless browser game using custom sprites, a scrolling background, a scoring system, and a conditional credits screen unlocked by performance.

A key architectural principle will be **Separation of Concerns**. Even though this will be a single-file application, the JavaScript code will be structured into logical objects or classes to ensure it is easy to read, debug, and modify.

## **2\. Technology Stack**

* **HTML5:** A single index.html file will provide the structure, primarily holding the \<canvas\> element which is the "screen" for our game.  
* **CSS3:** Embedded CSS (\<style\> tag) will be used to center the canvas, style the page background, and handle UI overlays. It will also import and apply a theme-appropriate font (e.g., **'Luckiest Guy' from Google Fonts**) to all UI text.  
* **JavaScript (ES6+):** All game logic will be written in vanilla JavaScript. We will use modern features like objects (or classes, as they are cleaner syntactically) to represent game entities.

## **3\. Visual & Thematic Design**

The game's aesthetic will be based on a "Legoland trip" theme, characterized by bright, primary colors and block-based visuals.

* **Typography:** All UI text (score, "Game Over", "Click to Start") will use the **'Luckiest Guy'** Google Font to evoke a fun, blocky, and playful feel. The fallback will be a standard rounded sans-serif (like Arial Rounded MT Bold or sans-serif).  
* **Color Palette:** The UI will favor a high-contrast palette of primary colors: bright red, yellow, blue, and green, set against white and black for clarity.  
* **UI Elements:** UI components like message boxes ("Game Over") and button prompts ("View Credits") will be styled to look as if they are constructed from blocks (e.g., using box-shadow and border to simulate 3D bricks).  
* **Assets (User-Provided):** This design assumes the user-provided assets will fit the theme:  
  * **Player:** A minifigure or similar block-based character.  
  * **Obstacles:** Custom sprites representing Lego creations (e.g., small trees, walls, blocky creatures).  
  * **Background:** A scrolling background image depicting a landscape built from bricks.

## **4\. Core Game Mechanics**

### **4.1. Player**

* The player (dino, or your custom sprite) will have a fixed horizontal (X) position on the left side of the canvas.  
* Its vertical (Y) position will be controlled by jumping.  
* **Input:** The player will jump when the user presses the Space key or taps the screen.  
* **Physics:** A simple gravity mechanic will pull the player down. Jumping applies an upward vertical velocity (vy), which is decreased by gravity in each frame. The player cannot jump again until they are on the ground.

### **4.2. Obstacles**

* Obstacles (your custom sprites) will be spawned off-screen to the far right.  
* They will move from right to left across the screen at the current gameSpeed.  
* Obstacles will be removed from memory once they move completely off-screen to the left to prevent performance issues.  
* Spawning will be semi-random, with a timer ensuring a minimum and maximum gap between obstacles.

### **4.3. Scrolling Background**

* Your custom background image will scroll continuously from right to left.  
* To create an "infinite" loop, we will draw the image twice, side-by-side. As the images scroll left, once the first image is completely off-screen, it will be reset to appear immediately after the second image. This gives a seamless, repeating effect.  
* The background's scroll speed will be a fraction of the gameSpeed to create a parallax effect (depth).

### **4.4. Scoring & Speed**

* The score will start at 0 and increase over time as long as the game is running.  
* The gameSpeed will also start at a base value and slowly increase with the score, making the game progressively harder.

### **4.5. Game States**

The game will be managed by a simple state machine. The primary states will be:

* LOADING: Initial state while assets (images) are being loaded.  
* READY: Assets are loaded. The game shows a "Click to Start" message.  
* PLAYING: The main game loop is active. Player can jump, obstacles spawn, score increases.  
* GAME\_OVER: Triggered by a collision. The game loop stops.  
* CREDITS: A special state entered from GAME\_OVER *only if* score \> CREDIT\_THRESHOLD.

### **4.6. Collision Detection (Updated)**

* We will use **Composite AABB (Compound Hitboxes)**. This provides a balance between performance and accuracy, ideal for convoluted shapes.  
* The **Player** will be defined by a single bounding box (a hitbox).  
* Each **Obstacle** will be defined by an *array* of one or more bounding boxes (its "collision map"). These sub-hitboxes are relative to the obstacle's main position.  
* A collision occurs if the Player's hitbox overlaps with *any* of the hitboxes in an Obstacle's array.  
* The core AABB check for two rectangles ( a and b, each with x, y, width, height) remains simple and fast:  
  return a.x \< b.x \+ b.width && a.x \+ a.width \> b.x && a.y \< b.y \+ b.height && a.y \+ a.height \> b.y;

### **4.7. Credits Screen**

* A constant, CREDIT\_THRESHOLD (e.g., 1000 points), will be defined.  
* When the state changes to GAME\_OVER, the game will check: if (score \> CREDIT\_THRESHOLD).  
* If true, the "Game Over" screen will display the score *and* an option to "View Credits".  
* Clicking this will change the state to CREDITS, which will display your custom credits text/images.

## **5\. Proposed Code Architecture (for Debugging)**

To make this easy to modify, we won't put all logic into one giant update() function. We'll use objects to manage distinct parts of the game.

* Game: The main object.  
  * properties: canvas, context, width, height, gameState, score, gameSpeed, player, obstacleManager, background.  
  * methods: init() (set up), loadAssets() (load images), start() (change state to PLAYING), gameLoop() (the main requestAnimationFrame loop), update() (call update on all sub-objects), draw() (call draw on all sub-objects), gameOver().  
* Player:  
  * properties: x, y, width, height, sprite, vy (vertical velocity), gravity, jumpStrength, isGrounded.  
  * **(Implicit hitbox):** Its x, y, width, height will serve as its single hitbox.  
  * methods: jump(), update(gameSpeed), draw(context), getHitbox() (returns its x,y,w,h).  
* ObstacleManager:  
  * properties: obstacles (an array). Each object in this array will contain its main x, y position, its sprite, and a hitboxes: \[\] array. The hitboxes array will contain objects with *relative* { x, y, width, height } properties.  
  * spriteImages (your collection), spawnTimer, minSpawnTime, maxSpawnTime.  
  * methods: spawnObstacle(), update(gameSpeed), draw(context),  
  * checkCollision(player): **(Updated)** This method will iterate through each obstacle on screen. For each obstacle, it will then iterate through that obstacle's hitboxes array. It performs an AABB check between the player.getHitbox() and each *absolute* hitbox position (obstacle x/y \+ relative hitbox x/y). If any check is true, it returns true.  
  * reset().  
* ScrollingBackground:  
  * properties: image, x1, x2, y, width, height, speedFactor.  
  * methods: update(gameSpeed), draw(context).  
* UI:  
  * methods:  
    * drawScore(score): Draws score using the 'Luckiest Guy' font.  
    * drawStartScreen(): Renders 'Click to Start' with the theme font.  
    * drawGameOverScreen(score, canShowCredits): Renders 'Game Over' text and score, styled in a theme-appropriate "brick" panel.  
    * drawCreditsScreen(): Renders credits text, also styled with theme font.

## **6\. Asset Handling**

1. We will define a list of asset URLs at the top of the script (e.g., const backgroundImageURL \= '...';, const obstacleSpriteURLs \= \['...', '...'\];).  
2. The Game.loadAssets() method will create new Image() objects for each.  
3. It will use Promise.all to wait for all image onload events to fire.  
4. Only when all images are loaded will the gameState change from LOADING to READY. This prevents the game from starting with broken/missing images.

## **7\. Implementation Plan (Single File)**

The index.html file will be structured as follows:

\<\!DOCTYPE html\>  
\<html\>  
\<head\>  
    \<title\>My Game\</title\>  
    \<style\>  
        /\* CSS to center canvas, style body \*/  
        /\* CSS to import @font-face 'Luckiest Guy' \*/  
    \</style\>  
\</head\>  
\<body\>  
    \<canvas id="gameCanvas"\>\</canvas\>

    \<script\>  
        // \--- 1\. CONFIGURATION \---  
        // (Constants like GRAVITY, JUMP\_STRENGTH, CREDIT\_THRESHOLD)

        // \--- 2\. ASSET DEFINITIONS \---  
        // (URLs for your background and obstacle sprites)

        // \--- 3\. CLASS/OBJECT DEFINITIONS \---  
        // (Player object/class)  
        // (Obstacle object/class. This class will be responsible for  
        //  holding its x, y, sprite, and its relative hitboxes array)  
        // (ObstacleManager object/class)  
        // (ScrollingBackground object/class)  
        // (UI object)

        // \--- 4\. GAME INITIALIZATION \---  
        // (The main Game object definition and init method)

        // \--- 5\. EVENT LISTENERS \---  
        // (window.addEventListener for 'keydown' and 'click'/'touchstart')

        // \--- 6\. START GAME \---  
        // (e.g., const myGame \= new Game(); myGame.loadAssets();)  
    \</script\>  
\</body\>  
\</html\>

## **8\. Next Steps**

1. **Approval:** Please review this updated plan.  
2. **Assets:** If you approve, you will need to provide the URLs for your custom sprites and background image. We can use placeholders initially if you prefer.  
3. **Implementation:** I will proceed to write the complete, single-file game code based on this approved architecture.
