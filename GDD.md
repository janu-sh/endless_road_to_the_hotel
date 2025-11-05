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

### **4.1. Player & Sidekick Characters**

#### **Hero Player**
* The player (hero character) starts at a fixed horizontal position on the left side of the screen.  
* Its vertical (Y) position will be controlled by jumping.  
* **Input:** The player will jump when the user presses the Space key or taps the screen. The player can duck by pressing the DOWN arrow key.  
* **Physics:** A simple gravity mechanic will pull the player down. Jumping applies an upward vertical velocity (vy), which is decreased by gravity in each frame. The player cannot jump again until they are on the ground.
* **Progressive Movement:** The player slowly moves towards the boss (right) as the score increases, creating a dramatic "catching up" effect.

#### **Sidekick Characters**
* **Count:** 2 sidekick characters positioned to the left of the hero (at -80px and -160px offsets).
* **Visual Design:** Sidekicks use a different color scheme (purple) and are rendered at 80% opacity to distinguish them from the hero.
* **Behavior:** Sidekicks act as "shadows" of the hero, mirroring all actions with a delay:
  * First sidekick: 300ms delay
  * Second sidekick: 600ms delay
* **Action Mirroring:** When the hero jumps, ducks, or stands up, sidekicks perform the same action after their respective delays, creating the appearance of independently reacting to obstacles.
* **Technical Implementation:** 
  * Sidekicks are **visual-only sprites** (not physics sprites) to prevent flickering and physics conflicts.
  * An **action queue system** records all hero actions with timestamps.
  * Each sidekick processes the queue with its specific delay, marking actions as processed to prevent duplicate execution.
  * Manual jump physics simulate gravity for smooth visual jumping without physics engine overhead.
  * Sidekicks maintain their relative position to the hero as the hero moves across the screen.

### **4.2. Boss Character**

#### **Visual Design & Position**
* The boss character is positioned on the right side of the screen (starting at X: 870, moving towards X: 700).
* Rendered with depth priority (setDepth: 10) to appear above obstacles.
* Anchored to bottom-center (origin 0.5, 1) to stand on the ground level.

#### **Boss Behavior & Movement**
* **Progressive Movement:** The boss slowly moves towards the player (left) as the score increases, creating a "closing the gap" effect.
  * Initial position: `BOSS_INITIAL_X = 870` (right side)
  * Target position: `BOSS_TARGET_X = 700`
  * Movement speed: `BOSS_MOVEMENT_SPEED = 0.001` (slower than player for asymmetric dramatic effect)
* **Movement Formula:** `bossX = BOSS_INITIAL_X - (BOSS_INITIAL_X - BOSS_TARGET_X) * progressRatio`

#### **Boss Actions & Obstacle Spawning**

The boss is the source of all obstacles in the game, creating them through two distinct actions:

**1. Kicking (Ground Obstacles)**
* **Visual Feedback:** Boss texture changes to `BOSS_KICK` for 300ms
* **Spawn Behavior:** 
  * Ground obstacles spawn near the boss's feet (80px to the left of boss position)
  * Obstacles are anchored to bottom-center and placed at ground level
  * Two types of ground obstacles:
    * Small obstacle: 40x80px (hitbox: 35x75)
    * Large obstacle: 120x60px (hitbox: 115x55)
* **Player Response:** Player must jump over these obstacles

**2. Throwing (Flying Obstacles)**
* **Visual Feedback:** Boss texture changes to `BOSS_THROW` for 300ms
* **Spawn Behavior:**
  * Flying obstacles spawn near the boss's hand position (80px left of boss)
  * Spawned at 45px above ground level
  * Flying obstacle: 70x150px (hitbox: 70x150)
* **Player Response:** Player must duck under these obstacles

**Spawn Timing:**
* Initial spawn delay: 2000ms
* Subsequent delays: Random between 1500-3500ms, adjusted by game speed
* Spawn ratio: 2:1 ground to flying obstacles (ground obstacles more common)
* Delay formula: `delay = Phaser.Math.Between(1500, 3500) / (currentSpeed / initialSpeed)`

### **4.3. Obstacles**

* All obstacles are spawned by the boss character (see section 4.2).
* Obstacles move from right to left across the screen at the current gameSpeed.
* Obstacles are removed from memory once they move completely off-screen to the left to prevent performance issues.
* Each obstacle type has a custom hitbox configuration for precise collision detection.

### **4.4. Parallax Scrolling Background System**

#### **Multi-Layer Parallax**
The game uses a three-layer parallax scrolling system to create depth and visual interest:

**Layer 1 - Far Background (Sky/Mountains)**
* Scroll speed: `gameSpeed * 0.2` (slowest)
* Alpha: 0.8 (slightly transparent)
* Purpose: Creates distant horizon effect

**Layer 2 - Mid Background (Distant Trees)**
* Scroll speed: `gameSpeed * 0.5` (medium)
* Alpha: 0.9
* Purpose: Middle-distance scenery

**Layer 3 - Near Background (Ground Layer)**
* Scroll speed: `gameSpeed * 0.8` (fastest)
* Alpha: 1.0 (fully opaque)
* Purpose: Foreground ground texture

#### **Technical Implementation**
* Uses Phaser's `TileSprite` for seamless infinite scrolling
* Each layer is 1200x600px and tiles horizontally
* Layers set to `setScrollFactor(0)` to remain fixed relative to camera
* Different scroll speeds create parallax depth effect as game speed increases

### **4.5. Rendering Depth Order & Shadow System**

#### **Depth Layering**
Phaser renders objects based on their depth value (lower values render behind, higher values render in front). The game uses the following depth order from back to front:

| Depth | Layer | Description |
|-------|-------|-------------|
| -10 | Background Far | Sky/mountains layer (farthest back) |
| -9 | Background Mid | Distant trees layer |
| -8 | Background Near | Ground texture layer (nearest background) |
| 1 | Character Shadows | Dynamic shadows for player and sidekicks |
| 2 | Player & Sidekicks | Hero and sidekick characters |
| 0 | Obstacles | Ground and flying obstacles (default depth) |
| 5 | Rain Particles | Rain drop effects |
| 6 | Splash Particles | Water splash effects |
| 9 | Boss Shadow | Dynamic shadow for boss character |
| 10 | Boss Character | Boss sprite (rendered above all obstacles) |

**Key Design Decisions:**
* Backgrounds use negative depth values to ensure they're always behind gameplay elements
* Shadows render at depth 1, just above backgrounds but below characters (depth 2)
* Boss and boss shadow use higher depth values (9-10) to render above obstacles
* Rain effects use mid-range depth (5-6) to appear in front of backgrounds but behind characters

#### **Dynamic Shadow System**

The game features dynamic elliptical shadows that render beneath all characters on the ground plane, providing visual depth and helping players judge jump height.

**Shadow Characteristics:**
* **Shape:** Elliptical shadows drawn using Phaser graphics
* **Position:** Always locked to ground level (GROUND_Y), regardless of character height
* **Dynamic Sizing:** Shadows shrink as characters jump higher
  * At ground level: 100% of base size
  * At maximum jump height (~200px): 30% of base size
  * Shrink factor formula: `max(0.3, 1 - (heightAboveGround / 400))`
* **Dynamic Transparency:** Shadow alpha scales with height
  * Base alpha: 0.3 (semi-transparent black)
  * Alpha formula: `0.3 * shrinkFactor`
  * Effect: Shadows become more transparent when characters are higher

**Shadow Base Sizes:**
* **Player Shadow:** 50px wide × 12px tall
* **Sidekick Shadows:** 45px wide × 10px tall (slightly smaller than player)
* **Boss Shadow:** 60px wide × 15px tall (larger to match boss size)

**Technical Implementation:**
```javascript
updateShadow(shadowGraphics, characterX, characterY, baseWidth, baseHeight) {
    const heightAboveGround = GROUND_Y - characterY;
    const shrinkFactor = Math.max(0.3, 1 - (heightAboveGround / 400));
    const shadowWidth = baseWidth * shrinkFactor;
    const shadowHeight = baseHeight * shrinkFactor;
    const shadowAlpha = 0.3 * shrinkFactor;
    
    shadowGraphics.clear();
    shadowGraphics.fillStyle(0x000000, shadowAlpha);
    shadowGraphics.fillEllipse(characterX, GROUND_Y, shadowWidth, shadowHeight);
}
```

**Benefits:**
* Provides visual feedback for jump height and landing position
* Enhances depth perception in the 2D game space
* Creates more polished, professional appearance
* Helps players time jumps and landings more accurately

### **4.6. Progressive Zoom & Camera System**

#### **Dynamic Camera Zoom**
The game features a progressive zoom system that creates increasing tension as the player advances:

**Zoom Parameters:**
* Initial zoom: `1.0` (normal view)
* Maximum zoom: `2.0` (2x magnification)
* Zoom increment: `0.001` per score point
* Zoom formula: `currentZoom = min(MAX_ZOOM, INITIAL_ZOOM + (score * ZOOM_INCREMENT))`

**Camera Configuration:**
* Origin point: Bottom-center (0.5, 1)
* Effect: Zoom expands upward from ground level, keeping the ground plane stable
* Purpose: Creates cinematic "closing in" effect as player and boss converge

#### **Camera Panning**
As zoom increases, the camera pans to maintain proper framing:

* Pan calculation: `cameraOffsetX = zoomProgress * 125`
* Where: `zoomProgress = (currentZoom - INITIAL_ZOOM) / (MAX_ZOOM - INITIAL_ZOOM)`
* Effect: Camera shifts right to keep both player and boss visible as they move closer
* Y-scroll remains at 0 to keep ground level at bottom of screen

#### **Visual Result**
The combined effect of:
1. Player moving right
2. Boss moving left
3. Progressive zoom in
4. Camera panning

Creates a dramatic "collision course" feeling where the player appears to be catching up to the boss while the view becomes more intense and focused.

### **4.7. Scoring & Speed**

* The score starts at 0 and increases over time (delta * 0.01) as long as the game is running.
* The gameSpeed starts at `INITIAL_GAME_SPEED = 8` and increases based on score.
* Speed formula: `gameSpeed = INITIAL_GAME_SPEED + (score * GAME_SPEED_INCREMENT)`
* Speed increment: `0.005` per score point
* Effect: Game progressively becomes harder as obstacles move faster and spawn more frequently.

### **4.8. Game States**

The game will be managed by a simple state machine. The primary states will be:

* LOADING: Initial state while assets (images) are being loaded.  
* READY: Assets are loaded. The game shows a "Click to Start" message.  
* PLAYING: The main game loop is active. Player can jump, obstacles spawn, score increases.  
* GAME\_OVER: Triggered by a collision. The game loop stops.  
* CREDITS: A special state entered from GAME\_OVER *only if* score \> CREDIT\_THRESHOLD.

### **4.9. Collision Detection (Updated)**

* We will use **Composite AABB (Compound Hitboxes)**. This provides a balance between performance and accuracy, ideal for convoluted shapes.  
* The **Player** will be defined by a single bounding box (a hitbox).  
* Each **Obstacle** will be defined by an *array* of one or more bounding boxes (its "collision map"). These sub-hitboxes are relative to the obstacle's main position.  
* A collision occurs if the Player's hitbox overlaps with *any* of the hitboxes in an Obstacle's array.  
* The core AABB check for two rectangles ( a and b, each with x, y, width, height) remains simple and fast:  
  return a.x \< b.x \+ b.width && a.x \+ a.width \> b.x && a.y \< b.y \+ b.height && a.y \+ a.height \> b.y;

### **4.10. Credits Screen**

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

## **8\. Technical Considerations & Lessons Learned**

### **8.1. Sidekick Implementation Challenges**

#### **Problem: Sprite Flickering**
During initial implementation, the leftmost sidekick character exhibited flickering behavior at game start that would eventually stabilize.

**Root Causes Identified:**
1. **Physics Engine Conflicts:** Initially, sidekicks were implemented as physics sprites with collision bodies. When changing textures and body sizes during duck/standup actions, the physics engine caused jittery repositioning.
2. **Action Processing Window:** The original time window check (`timeSinceAction >= delay && timeSinceAction < delay + 50`) allowed actions to be triggered multiple times across frames, causing rapid texture switching.

**Solutions Applied:**
1. **Visual-Only Sprites:** Converted sidekicks from `physics.add.sprite` to regular `add.sprite` objects, removing physics body interactions entirely.
2. **Action Tracking System:** Implemented a `processedBy` array on each action to track which sidekicks have executed it, preventing duplicate processing.
3. **Manual Jump Physics:** Created custom jump animation using manual Y-position updates with gravity calculations, avoiding physics engine overhead.
4. **Removed Time Window Upper Bound:** Changed from time window check to single-point execution check, ensuring each action triggers exactly once per sidekick.

**Key Takeaway:** For visual-only follower characters that mirror player actions, avoid physics bodies entirely. Use manual animation and state tracking for smoother, more predictable behavior.

### **8.2. Action Queue Architecture**

The action queue system provides a clean separation between hero input and sidekick behavior:

```javascript
// Action structure
{
    type: 'jump' | 'duck' | 'standup',
    time: timestamp,
    processedBy: [sidekickIndex, ...]
}
```

**Benefits:**
- Decouples hero actions from sidekick responses
- Allows different delays per sidekick
- Automatic cleanup of old actions (>1 second)
- Scalable to additional sidekicks without code changes