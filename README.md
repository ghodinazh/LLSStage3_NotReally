Touhou 4 Stage 3 remake (WinAPI, C++)

A small bullet hell game written in pure C++ with WinAPI and GDI. You play Marisa Kirisame and fight through the gateway to Mugenkan: mob waves, the Dark Mirror midboss, then the gatekeeper Elly, with dialogue before and after the boss.


ABOUT THIS PROJECT

1. Why the text is in Japanese

The dialogue is written in Japanese on purpose. I use this game to train my Japanese. Reading the lines while I play helps me connect new words and phrases to scenes I remember from the game, so they stay in my head better. The dialogue is stored in dialogue.txt (UTF-8), so you can edit or translate it freely.

2. This is not a copy of the original game

The project started as my own independent danmaku shooter with my own scenario. Later I wanted to recreate it as a remake of Touhou 4: Lotus Land Story, and in the end I mixed the two. The setting, the characters and the general structure of the stage come from Touhou 4, but the story lines, the numbers, the timings and the bullet patterns are my own interpretation. It does not try to reproduce the original game exactly.

3. Why sprites have purple marks on the edges

The game treats pure magenta (255, 0, 255) as the transparent color. My photo editor blends the edges of the sprites with their background when it saves them, so some edge pixels end up as a purple or pink shade instead of exact magenta. Those pixels are not made transparent, so you see purple marks around the characters. This is a problem with my photo editor that I could not fix. It is only a visual flaw of the sprites and it does not affect gameplay. If a sprite looks wrong, the game draws a simple colored shape instead when the image file is missing.


HOW TO LAUNCH THE GAME

open the .exe file


CONTROLS

Arrow keys: move
Shift (hold): focus, slower movement, shows the hitbox dot
Z: shoot (hold) and advance dialogue
X: bomb
Ctrl (hold): skip dialogue
Enter: restart after clear or game over
Esc: quit


TECHNICAL NOTES

Window 640x480, playfield 384x448, fixed 60 updates per second driven by QueryPerformanceCounter, double buffered drawing, sprites drawn with TransparentBlt. Music uses the Windows MCI interface (winmm).


CODE OVERVIEW

How the code was created

The whole game is one source file, main.cpp, written in plain C++ with the WinAPI. It was built up in layers: first the window and the fixed-speed game loop, then drawing with double buffering, keyboard input and Marisa, then bullets and mob enemies, then the stage script, the midboss, the boss, dialogue, sprite loading with fallback shapes, music, and finally Japanese text support.

Headers used

windows.h: the WinAPI itself (windows, messages, drawing, input, files).
algorithm: std::min and std::max.
cmath: sine, cosine, arctangent, absolute value and remainder for movement and aiming.
cstdio: snprintf for building text.
cstdlib: rand and srand for random numbers.
cstring: memcpy, strlen, strcmp for keys and strings.
fstream: reading dialogue.txt.
string and vector: text storage and the lists of bullets, enemies, shots, items and dialogue lines.

Libraries linked

user32.lib: windows and input.
gdi32.lib: drawing with GDI.
msimg32.lib: TransparentBlt for sprites with a transparent color.
winmm.lib: timeBeginPeriod for a precise Sleep, and mciSendString for music.

Main WinAPI tools used

CreateWindowExA, RegisterClassExA, PeekMessageA and DispatchMessageA: the window and message loop.
QueryPerformanceCounter and QueryPerformanceFrequency: the fixed 60 updates per second timer.
CreateCompatibleDC and CreateCompatibleBitmap: the two off-screen buffers (playfield and whole window) used for double buffering.
BitBlt: copying the finished picture to the window in one step, which removes flicker and tearing.
LoadImageA and TransparentBlt: loading BMP files and drawing them with magenta as the transparent color.
GetAsyncKeyState: reading the keyboard.
CreateSolidBrush, CreatePen, Ellipse, Rectangle, Polygon, Pie, FillRect: the fallback shapes drawn when a sprite file is missing, and the HUD.
TextOutA, TextOutW, DrawTextW, CreateFontA, CreateFontW: text. Japanese dialogue is converted from UTF-8 with MultiByteToWideChar and drawn with a Japanese capable font.
mciSendStringA: playing, stopping and looping the music.

Constants

Window 640x480, playfield 384x448 placed at x 128, y 16, magenta transparent color, bullet hit radii, bullet colors (8), boss HP per phase (1500, 2000, 2600) and time limit per phase (30 s, 30 s, 40 s).

Data structures

Egara: one loaded sprite sheet (device context, bitmap, size, and a flag that says if it was loaded).
Tama: one enemy bullet (position, speed, angle, type, color, gravity for tiles, graze flag).
Teki: one mob enemy (position, speed, HP, type, timer).
Hoshi: one shot fired by Marisa.
Aitemu: one item (power, point or bomb).
Kaiwa: one dialogue line (speaker and UTF-8 text).
Kagami: the Dark Mirror midboss.
Erii: the boss Elly (position, target position, HP, attack phase, timers, animation).
Sekai: the whole game state: player position, lives, bombs, power, score, graze, timers, key states and all the lists.
Joutai (enum): the game state: stage, dialogue before the boss, boss fight, dialogue after, clear, game over.

Functions (all names are Romaji)

Helpers
saikoroNoYuragi: random decimal number in a range.
futariNoKakudo: angle from one point to another, used for aimed bullets.
tsukimiDangoNoSaisho: true only on the frame a key was first pressed.
kuukiNoSaikai: reads the keyboard each frame (ignored when the window is not active).
ikiteiruMonoDake: template that removes dead objects from a list.

World and sprites
sekaiWoUmareKawaru: resets everything for a new game and starts the stage music.
egaraWoKamiNiYomu: loads one BMP file.
egaraNoKoma: draws one frame of a sprite sheet with transparency; returns false if the sprite was not loaded so the caller can draw a fallback.
katachiWoNuru: draws a filled rectangle or ellipse.
kotobaWoKaku: draws a text line.

Creating objects
tamaWoHanatsuYoukai: creates an enemy bullet.
kamaWoNageruOni: creates a boomerang scythe.
tekiWoYobiDasuTanuki: creates a mob enemy.
hoshiNoYaWoUtsu: creates a player shot.
aitemuWoBaraMaku: creates an item.
tamaWoKeshiSaru: removes all enemy bullets.

Damage and death
tekiNoSaigo: mob enemy dies, gives score and may drop an item.
kagamiNiDamage and kagamiNoSaigo: damage to the midboss and its defeat (drops items and a bomb).
eriiNiDamage: damage to Elly (ignored while she is invincible).
eriiNoDanWoKirikaeru: switches Elly to the next attack phase, or ends the fight and starts the closing dialogue.
jibunGaYarareta: Marisa is hit: loses a life, resets bombs, clears bullets, or game over.

Player
hoshiNoKawariNiMahou: Marisa's shot pattern (main shots plus missiles that grow with power level and narrow in focus mode).
mahouNoSupaaku: the bomb beam: clears bullets and damages enemies in front of Marisa.
marisaNoHashiri: movement (3.6 normal, 1.6 focused), shooting, bomb start and the short death-bomb window.

Updating the game
tekiNoKoushin: moves mobs and makes them shoot (three enemy types).
kagamiNoKoushin: Dark Mirror: moves, and fires aimed fans and rings from itself and its mirrored twin.
eriiNoKoushin: Elly: tile explosions while invincible, then three phases with scythes, rice grain fans, spirals, rings and falling bullets.
tamaNoKoushin: moves bullets, handles scythe return flight and tile gravity, checks hits and grazes.
hoshiNoKoushin: moves player shots and checks hits on enemies and bosses.
aitemuNoKoushin: moves items, auto-collects them when Marisa is near the top or bombing, and applies pickups.
sutejiNoDenpou: the stage script: spawns waves by timer, starts the midboss at 30 seconds and the boss dialogue at the end.
kaiwaNoSusumi: advances dialogue with Z, Enter or Ctrl and switches state when it ends.
subeteWoKoushin: one game tick: input, scrolling, all updates and the music loop check.

Drawing
haikeiWoNuru: scrolling background (the BMP, or a scrolling checkered tile floor).
marisaNoMigawari, ayaNoMigawari, kagamiNoMigawari, eriiNoMigawari: simple fallback figures for Marisa, mobs, Dark Mirror and Elly.
tamaWoNuru: draws one bullet (sprite or fallback shape).
bashoWoNuru: draws the whole playfield: background, items, enemies, bosses, Marisa, shots, bomb beam, bullets, hitbox dot, boss life bar and messages.
subeteWoEgaku: draws the side panels and score, then copies the playfield into the window buffer.

Start-up
kamiNoKagaribi: loads all sprite files from the assets folder.
doguWoKaiAtsumeru: creates brushes, pens and fonts.
yoruNoMadoNoTobira: window procedure: handles closing, painting and the Esc key.
WinMain: creates the window and buffers, loads everything, runs the loop (update at a fixed 60 times per second, draw as often as possible), then cleans up.

Stage timeline (in game seconds at 60 updates per second)
1 to 10 s: fairies from both sides with aimed rice grain bullets.
12 to 21 s: slow fairies that fire rings.
22 to 29 s: fast diving fairies.
30 s: Dark Mirror midboss (limit about 25 s).
After it: more fairy waves, then the boss dialogue at 55 s.
Elly: invincible tile opening (4.5 s), then three attack phases.

Known limits

Bullet patterns, speeds, HP values and timings are my own approximation, not data from the original game. Marisa has one shot type only. There is no title screen, no pause, no high score saving and no sound effects.
