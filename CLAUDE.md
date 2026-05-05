# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-file 3D RPG game built with Three.js. The entire game is contained in `index.html` - a self-contained HTML file with embedded CSS and JavaScript.

## Architecture

### Single-File Structure
The game uses a monolithic architecture where all code exists in one HTML file:
- **HTML**: UI elements (HUD, modals, overlays)
- **CSS**: Styling for UI components
- **JavaScript**: Game logic, Three.js 3D rendering, game systems

### Core Game Systems

**3D Rendering (Three.js)**
- Scene management with camera, renderer, lights
- Player model: humanoid with body, head, arms, legs
- Enemy models: unique 3D models per type (Slimes, Goblins, Orcs, Dragons, Demons)
- Item models: detailed 3D representations (weapons, potions, coins, gems)
- Terrain: procedurally varied ground with sky sphere

**Player System**
- `playerStats` object: level, HP, mana, XP, strength, gold, weapon, skillPoints
- `playerSkills` object: vitality, endurance, power, agility, fortune, regeneration
- Animation system: walking, running, idle poses with limb movement
- Movement: WASD controls, sprint (Shift), jump (Space)
- Combat: melee attacks (LKM) and 4 magic spells (Q/E/R/F)

**Enemy AI**
- Pathfinding: enemies move toward player using vector math
- Type-specific animations (slime bounce, dragon wing flap, demon rotation)
- Night buff: 20% speed increase during night time
- Boss mechanics: larger size, more HP, special effects

**Game Systems**
- **Save/Load**: localStorage persistence (auto-save every 30s)
- **Day/Night Cycle**: `dayTime` variable (0-1), dynamic lighting and sky colors
- **Quest System**: `activeQuests` array with kill/collect/level objectives
- **Achievement System**: `unlockedAchievements` array with unlock conditions
- **Shop System**: NPC merchant with purchasable items and upgrades
- **Skill System**: 6 skill trees with passive bonuses
- **Inventory**: 5-slot system with consumables and equipment

**Collision Detection**
- Uses Three.js `Box3` for AABB collision checks
- Applied to: player-enemy, player-item, player-chest, player-shopkeeper

**Particle System**
- Custom particle spawning for visual effects
- Used for: attacks, spell casts, item collection, level ups

### Key Global Variables

```javascript
// Core objects
let scene, camera, renderer, player, ground
let enemies = []      // Array of enemy meshes
let items = []        // Array of item meshes
let chests = []       // Array of chest groups
let particles = []    // Array of particle effects
let shopkeeper        // NPC merchant mesh

// Player state
let playerStats = { level, hp, maxHp, mana, maxMana, xp, maxXp, strength, gold, weapon, hasShield, skillPoints }
let playerSkills = { vitality, endurance, power, agility, fortune, regeneration }
let inventory = []    // Array of collected items

// Game state
let gameActive = true
let dayTime = 0       // 0-1 value for day/night cycle
let gameStats = { killCount, itemCount, startTime }
let unlockedAchievements = []
let activeQuests = []
```

### Data Structures

**Item Types**: `itemTypes` object defines all collectible items with properties:
- Consumables: HEALTH_POTION, STRENGTH_POTION
- Currency: COIN, GEM
- Weapons: SWORD, AXE, BOW, STAFF (with damage and speed stats)

**Enemy Types**: `enemyTypes` object defines enemy properties:
- Stats: hp, xp, gold, speed, size
- Boss flag: `isBoss` boolean
- Types: SLIME, GOBLIN, ORC, DRAGON, DEMON

**Spell Types**: `spellTypes` object defines magic spells:
- Properties: damage/heal, cooldown, manaCost
- Types: FIREBALL, HEAL, LIGHTNING, SHIELD

### Main Game Loop

The `animate()` function runs every frame:
1. `updatePlayer()` - movement, animation, collision
2. `updateEnemies()` - AI pathfinding, animations
3. `checkItemCollection()` - pickup detection
4. `checkChestInteraction()` - chest opening
5. `checkShopkeeperInteraction()` - merchant proximity
6. `updateParticles()` - particle lifecycle
7. `drawMinimap()` - 2D map rendering
8. `updateDayNightCycle()` - lighting and sky updates
9. Mana/HP regeneration based on skills
10. `renderer.render(scene, camera)` - final render

### UI Modals

Three modal systems with overlay pattern:
- **Shop Modal**: `#shopModal` + `#shopOverlay` - merchant interface
- **Skills Modal**: `#skillsModal` + `#skillsOverlay` - skill tree interface
- **Game Over Modal**: `#gameOver` - death screen with restart

### Development Notes

**Adding New Features**:
- New items: add to `itemTypes` object and `spawnItem()` switch
- New enemies: add to `enemyTypes` object and `spawnEnemy()` switch with custom model
- New spells: add to `spellTypes` object and create `cast[SpellName]()` function
- New skills: add to `skillsData` and `playerSkills`, implement effect in relevant functions

**Testing**:
- Open `index.html` in browser (no build step required)
- Use browser DevTools console for debugging
- Check localStorage for save data: `localStorage.getItem('rpg_save')`

**Performance Considerations**:
- Particle cleanup happens automatically when `life <= 0`
- Enemies respawn 2 seconds after death
- Items respawn 3 seconds after collection
- Auto-save throttled to 30-second intervals

**Skill Effects Implementation**:
- Vitality/Endurance/Power: applied immediately in `upgradeSkill()`
- Agility: multiplier applied in `updatePlayer()` to movement speed
- Fortune: multiplier applied in `gainGold()` to gold rewards
- Regeneration: passive regen in main `animate()` loop

**Day/Night Cycle**:
- `dayTime` increments by `daySpeed` (0.0001) each frame
- Sky color interpolation using `lerpColor()` function
- Sun position calculated with trigonometry based on `dayTime`
- Enemy speed buff applied/removed based on `isDay` boolean

## Deployment

The game is deployed via GitHub Pages:
- Repository: https://github.com/dimdimych96/3d-rpg-game
- Live URL: https://dimdimych96.github.io/3d-rpg-game/
- Deployment: automatic on push to `main` branch

To update the live game:
```bash
git add index.html
git commit -m "Description of changes"
git push origin main
```

Changes appear live within 1-2 minutes after push.
