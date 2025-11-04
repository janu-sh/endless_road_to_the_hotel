# TODO: Animation Implementation

## Overview
The current game uses static placeholder images. This document outlines the steps to implement sprite sheet animations for player and boss characters.

## Current State
- **Player**: Static images for idle and duck states
- **Boss**: Static images for idle, kick, and throw states
- **Sidekicks**: Static images mirroring player states
- **Loading**: Uses `this.load.image()` for single images

## Required Animations

### Player Character
- **Idle/Walking**: Looping animation for standing/running state
- **Ducking**: Animation for ducked state (can be looping or static)
- **Jumping**: Optional - frames for jump arc (start, mid, end)

### Boss Character
- **Idle/Walking**: Looping animation for default state
- **Kicking**: One-shot animation when spawning ground obstacles
- **Throwing**: One-shot animation when spawning flying obstacles

### Sidekick Characters
- Will automatically mirror player animations through existing action queue system

## Implementation Steps

### 1. Create Sprite Sheets
Create sprite sheet images with the following specifications:

**Player Sprite Sheet** (`player_spritesheet.png`)
- Frame size: 60x80 pixels
- Suggested layout:
  - Frames 0-3: Idle/walking animation
  - Frames 4-7: Duck animation
  - Frames 8-11: Jump animation (optional)

**Boss Sprite Sheet** (`boss_spritesheet.png`)
- Frame size: 120x180 pixels
- Suggested layout:
  - Frames 0-5: Idle/walking animation
  - Frames 6-10: Kick animation
  - Frames 11-15: Throw animation

**Sidekick Sprite Sheet** (`sidekick_spritesheet.png`)
- Frame size: 60x80 pixels
- Same layout as player (different color scheme - purple)

### 2. Update Asset Definitions (Lines 106-159)

Replace current ASSETS object with sprite sheet definitions:

```javascript
const ASSETS = {
    PLAYER_SPRITESHEET: { 
        key: 'player_sheet', 
        url: 'assets/player_spritesheet.png',
        frameWidth: 60,
        frameHeight: 80
    },
    SIDEKICK_SPRITESHEET: {
        key: 'sidekick_sheet',
        url: 'assets/sidekick_spritesheet.png',
        frameWidth: 60,
        frameHeight: 80
    },
    BOSS_SPRITESHEET: {
        key: 'boss_sheet',
        url: 'assets/boss_spritesheet.png',
        frameWidth: 120,
        frameHeight: 180
    },
    // Keep existing background and obstacle assets
    BACKGROUND_FAR: { ... },
    BACKGROUND_MID: { ... },
    BACKGROUND_NEAR: { ... },
    OBSTACLE_SMALL: { ... },
    OBSTACLE_LARGE: { ... },
    OBSTACLE_FLYING: { ... }
};
```

### 3. Update BootScene.preload() (Lines 196-211)

Replace image loading with spritesheet loading:

```javascript
preload() {
    console.log('BootScene: preload');
    
    // Load sprite sheets
    this.load.spritesheet(
        ASSETS.PLAYER_SPRITESHEET.key, 
        ASSETS.PLAYER_SPRITESHEET.url,
        { 
            frameWidth: ASSETS.PLAYER_SPRITESHEET.frameWidth, 
            frameHeight: ASSETS.PLAYER_SPRITESHEET.frameHeight 
        }
    );
    
    this.load.spritesheet(
        ASSETS.SIDEKICK_SPRITESHEET.key, 
        ASSETS.SIDEKICK_SPRITESHEET.url,
        { 
            frameWidth: ASSETS.SIDEKICK_SPRITESHEET.frameWidth, 
            frameHeight: ASSETS.SIDEKICK_SPRITESHEET.frameHeight 
        }
    );
    
    this.load.spritesheet(
        ASSETS.BOSS_SPRITESHEET.key, 
        ASSETS.BOSS_SPRITESHEET.url,
        { 
            frameWidth: ASSETS.BOSS_SPRITESHEET.frameWidth, 
            frameHeight: ASSETS.BOSS_SPRITESHEET.frameHeight 
        }
    );
    
    // Keep existing background and obstacle image loading
    this.load.image(ASSETS.BACKGROUND_FAR.key, ASSETS.BACKGROUND_FAR.url);
    this.load.image(ASSETS.BACKGROUND_MID.key, ASSETS.BACKGROUND_MID.url);
    this.load.image(ASSETS.BACKGROUND_NEAR.key, ASSETS.BACKGROUND_NEAR.url);
    this.load.image(ASSETS.OBSTACLE_SMALL.key, ASSETS.OBSTACLE_SMALL.url);
    this.load.image(ASSETS.OBSTACLE_LARGE.key, ASSETS.OBSTACLE_LARGE.url);
    this.load.image(ASSETS.OBSTACLE_FLYING.key, ASSETS.OBSTACLE_FLYING.url);
    
    // Loading text
    const loadingStyle = { font: "32px Arial", fill: "#FFFFFF" };
    this.loadingText = this.add.text(GAME_WIDTH / 2, GAME_HEIGHT / 2, "LOADING...", loadingStyle).setOrigin(0.5);
}
```

### 4. Create Animations in GameScene.create() (After line 297)

Add animation definitions after background setup:

```javascript
// --- Animation Definitions ---

// Player Animations
this.anims.create({
    key: 'player_idle',
    frames: this.anims.generateFrameNumbers(ASSETS.PLAYER_SPRITESHEET.key, { start: 0, end: 3 }),
    frameRate: 8,
    repeat: -1
});

this.anims.create({
    key: 'player_duck',
    frames: this.anims.generateFrameNumbers(ASSETS.PLAYER_SPRITESHEET.key, { start: 4, end: 7 }),
    frameRate: 8,
    repeat: -1
});

this.anims.create({
    key: 'player_jump',
    frames: this.anims.generateFrameNumbers(ASSETS.PLAYER_SPRITESHEET.key, { start: 8, end: 11 }),
    frameRate: 10,
    repeat: 0
});

// Sidekick Animations (same structure, different sprite sheet)
this.anims.create({
    key: 'sidekick_idle',
    frames: this.anims.generateFrameNumbers(ASSETS.SIDEKICK_SPRITESHEET.key, { start: 0, end: 3 }),
    frameRate: 8,
    repeat: -1
});

this.anims.create({
    key: 'sidekick_duck',
    frames: this.anims.generateFrameNumbers(ASSETS.SIDEKICK_SPRITESHEET.key, { start: 4, end: 7 }),
    frameRate: 8,
    repeat: -1
});

this.anims.create({
    key: 'sidekick_jump',
    frames: this.anims.generateFrameNumbers(ASSETS.SIDEKICK_SPRITESHEET.key, { start: 8, end: 11 }),
    frameRate: 10,
    repeat: 0
});

// Boss Animations
this.anims.create({
    key: 'boss_idle',
    frames: this.anims.generateFrameNumbers(ASSETS.BOSS_SPRITESHEET.key, { start: 0, end: 5 }),
    frameRate: 10,
    repeat: -1
});

this.anims.create({
    key: 'boss_kick',
    frames: this.anims.generateFrameNumbers(ASSETS.BOSS_SPRITESHEET.key, { start: 6, end: 10 }),
    frameRate: 12,
    repeat: 0
});

this.anims.create({
    key: 'boss_throw',
    frames: this.anims.generateFrameNumbers(ASSETS.BOSS_SPRITESHEET.key, { start: 11, end: 15 }),
    frameRate: 12,
    repeat: 0
});
```

### 5. Update Player Creation (Line 306)

Change player sprite creation to use sprite sheet:

```javascript
// OLD:
this.player = this.physics.add.sprite(PLAYER_INITIAL_X, GROUND_Y, ASSETS.PLAYER.key)

// NEW:
this.player = this.physics.add.sprite(PLAYER_INITIAL_X, GROUND_Y, ASSETS.PLAYER_SPRITESHEET.key)
    .setOrigin(0.5, 1)
    .setCollideWorldBounds(true);

// Start idle animation
this.player.play('player_idle');
```

### 6. Update Sidekick Creation (Lines 322-330)

```javascript
const sidekick = this.add.sprite(
    PLAYER_INITIAL_X + sidekickOffsets[i], 
    GROUND_Y, 
    ASSETS.SIDEKICK_SPRITESHEET.key  // Changed from ASSETS.SIDEKICK.key
)
    .setOrigin(0.5, 1)
    .setAlpha(0.8);

// Start idle animation
sidekick.play('sidekick_idle');
```

### 7. Update Boss Creation (Line 346)

```javascript
this.boss = this.add.sprite(bossX, bossY, ASSETS.BOSS_SPRITESHEET.key)  // Changed
    .setOrigin(0.5, 1)
    .setDepth(10);

// Start idle animation
this.boss.play('boss_idle');
```

### 8. Replace setTexture() Calls with play()

**Player Duck (Line 419):**
```javascript
// OLD:
this.player.setTexture(ASSETS.PLAYER_DUCK.key);

// NEW:
this.player.play('player_duck');
```

**Player Stand Up (Line 438):**
```javascript
// OLD:
this.player.setTexture(ASSETS.PLAYER.key);

// NEW:
this.player.play('player_idle');
```

**Boss Kick Animation (Line 459):**
```javascript
// OLD:
this.boss.setTexture(ASSETS.BOSS_KICK.key);
this.time.delayedCall(300, () => {
    if (this.boss) this.boss.setTexture(ASSETS.BOSS.key);
});

// NEW:
this.boss.play('boss_kick');
this.boss.once('animationcomplete', () => {
    if (this.boss) this.boss.play('boss_idle');
});
```

**Boss Throw Animation (Line 477):**
```javascript
// OLD:
this.boss.setTexture(ASSETS.BOSS_THROW.key);
this.time.delayedCall(300, () => {
    if (this.boss) this.boss.setTexture(ASSETS.BOSS.key);
});

// NEW:
this.boss.play('boss_throw');
this.boss.once('animationcomplete', () => {
    if (this.boss) this.boss.play('boss_idle');
});
```

**Sidekick Duck (Line 595):**
```javascript
// OLD:
sidekick.sprite.setTexture(ASSETS.SIDEKICK_DUCK.key);

// NEW:
sidekick.sprite.play('sidekick_duck');
```

**Sidekick Stand Up (Line 598):**
```javascript
// OLD:
sidekick.sprite.setTexture(ASSETS.SIDEKICK.key);

// NEW:
sidekick.sprite.play('sidekick_idle');
```

### 9. Optional: Add Jump Animation Trigger (Line 405)

```javascript
if (this.player.body.touching.down) {
    this.player.setVelocityY(PLAYER_JUMP_VELOCITY);
    this.player.play('player_jump');  // Add this
    
    // Record action for sidekicks
    this.actionQueue.push({
        type: 'jump',
        time: this.time.now
    });
}
```

And in sidekick jump handling (Line 590):
```javascript
if (action.type === 'jump' && !sidekick.isJumping) {
    sidekick.isJumping = true;
    sidekick.jumpVelocity = PLAYER_JUMP_VELOCITY;
    sidekick.sprite.play('sidekick_jump');  // Add this
}
```

## Testing Checklist

After implementation, verify:
- [ ] Player idle animation loops correctly
- [ ] Player duck animation plays when pressing DOWN
- [ ] Player returns to idle when releasing DOWN
- [ ] Boss idle animation loops correctly
- [ ] Boss kick animation plays when spawning ground obstacles
- [ ] Boss throw animation plays when spawning flying obstacles
- [ ] Boss returns to idle after kick/throw animations complete
- [ ] Sidekicks mirror all player animations with proper delays
- [ ] Collision detection still works correctly with animated sprites
- [ ] No performance issues with multiple animated sprites

## Notes

### Hitbox Considerations
- Sprite animations should maintain consistent dimensions (60x80 for player, 120x180 for boss)
- If animation frames have different visual sizes, ensure hitbox adjustments in duck() and standUp() methods still work correctly
- Test collision detection thoroughly with animated sprites

### Performance
- Phaser 3 handles sprite sheet animations efficiently
- Current game has 3 animated characters (player + 2 sidekicks) + 1 boss = minimal performance impact
- Obstacle sprites can remain static images unless animation is desired

### Future Enhancements
- Add hurt/death animation for player
- Add victory animation when reaching high scores
- Add particle effects on boss attacks
- Add animation for obstacles (e.g., spinning, flapping)

## Asset Creation Tips

### Sprite Sheet Layout
- Use a horizontal strip layout (all frames in one row) for simplicity
- Ensure consistent frame dimensions
- Leave no gaps between frames
- Use transparent backgrounds (PNG format)

### Animation Timing
- Idle animations: 6-10 fps for subtle movement
- Action animations (kick, throw): 10-15 fps for snappy actions
- Match animation duration to game feel (current boss actions are 300ms)

### Tools
- **Aseprite**: Excellent for pixel art sprite sheets
- **Piskel**: Free online pixel art tool
- **TexturePacker**: For combining individual frames into sprite sheets
- **Shoebox**: Free sprite sheet packer

## References
- [Phaser 3 Sprite Sheets Documentation](https://photonstorm.github.io/phaser3-docs/Phaser.Loader.LoaderPlugin.html#spritesheet)
- [Phaser 3 Animations Documentation](https://photonstorm.github.io/phaser3-docs/Phaser.Animations.AnimationManager.html)
