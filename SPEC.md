Boil Master — Project Specification

Overview

A mobile-first Progressive Web App (PWA) that guides a chef through a crawfish, shrimp, and/or lobster boil. 
The app manages ingredient selection, calculates quantities based on guest count, determines the correct add-order 
so everything finishes cooking at the same time, and provides a live timer with audio chimes at each add step.


Platform & Delivery


Delivered as a static file package (index.html, manifest.json, sw.js, icons/) hosted on GitHub Pages
Installable on iPhone via Safari → Share → Add to Home Screen
Runs as a standalone PWA (no browser chrome, full screen)
Works fully offline via service worker caching
All asset paths must be relative (./) to support GitHub Pages subpath hosting



App Structure

Three screens navigated sequentially. Back navigation is available from Screen 2 only.

Screen 1 — Ingredient Selection


Guest count selector displayed just below the header

Default: 4 guests
Stepper buttons (− / +), range 1–50
Changing guest count instantly updates all quantity badges



Ingredient list grouped into four sections: Base, Vegetables, Proteins, Extras
Each ingredient row displays:

Checkbox
Ingredient name
Short preparation note
Quantity badge (calculated from guest count, in red)
Cook time label (in minutes, or "At boil" for 0-minute items); items ≥ 15 min shown in red



Bottom bar shows selected count and a View Cooking Order → button (disabled until at least one ingredient is selected)


Screen 2 — Cooking Instructions


Displays total cook time (accounting for the 1-minute post-seasoning delay)
Ordered step cards showing:

Step number
Ingredient name(s) with calculated quantities
Preparation notes
When to add (e.g. "When water reaches a rolling boil", "At the 21-minute mark (1 min after starting timer)")
How long each item cooks



Ingredients sharing the same add-time are grouped into a single step
Start Cook 🔥 button at the bottom


Screen 3 — Active Timer


Elapsed time clock (MM:SS)
Pre-cook banner (yellow): appears immediately when Start Cook is tapped

Lists at-boil ingredients (seasoning, lemons, etc.) with quantities
"✓ Added — Start Timer!" button; tapping this starts the countdown
Timer does NOT start until the user taps this button



Alert banner (red): appears when each timed ingredient is due

Lists ingredient(s) with quantities to add
"✓ Added it!" dismiss button
Accompanied by an audio chime (4-note ascending tone)



Next up bar: shows the next ingredient and countdown to its add time
Live step list: all timed steps with running "minutes left" countdown
Chef note: "🍺 Note to Chef — cooking is hot work, stay hydrated with a cold beer!" displayed above the Stop Cook button
✕ Stop Cook button: stops timer and returns to Screen 1
All done banner (green): appears when the last ingredient has finished cooking



Ingredients

IDNameSectionCook TimeQty/PersonUnitRounding
seasoningCrawfish/boil seasoningBaseAt boil0.5tbsp0.5
lemonLemons (halved)BaseAt boil0.5—0.5
onionOnions (quartered)Base20 min0.25—0.25
garlicGarlic (whole heads)Base20 min0.25head0.25
potatoPotatoesVegetables20 min0.5lb0.25
carrotCarrots (chunked)Vegetables15 min0.5—1
cornCorn on the cobVegetables10 min0.5ear0.5
mushroomMushroomsVegetables5 min3oz1
sausageAndouille/smoked sausageProteins10 min3oz1
lobsterLobster (live)Proteins12 min0.5—0.5
crablegsCrab legsProteins8 min0.5lb0.25
crabBlue/Dungeness crab (whole)Proteins8 min0.5—0.5
clamClamsProteins6 min4—1c
rawfishLive crawfishProteins5 min1lb0.25
shrimpShrimp (shell-on)Proteins3 min0.33lb0.25
eggEggsExtras12 min1—1
artichokeArtichokes (halved)Extras20 min0.5—0.5
brusselBrussels sproutsExtras8 min3oz1


Cooking Order Logic


Add time is calculated as: addAtMin = maxCookTime − ingredient.cookTime
Items with cookTime = 0 always get addAtMin = 0 (added at the boil, before the timer starts)
When at-boil items are present, all timed step addAtMin values are shifted +1 minute to allow the water to return to a full boil after seasoning is added
Ingredients sharing the same addAtMin are grouped into a single step
Steps are sorted ascending by addAtMin



Quantity Calculation


rawQty = ingredient.qty × guests
displayQty = round(rawQty / round) × round, minimum of one round unit
Fractions displayed as unicode: ½, ¼, ¾
Quantities appear on ingredient rows, step cards, precook banner, and alert banners



Audio (iOS-compatible)


AudioContext is created and unlocked on the user's first tap anywhere in the app (required by iOS)
A silent buffer is played at unlock time to fully authorize the context
The context is reused for all subsequent chimes; suspended contexts are resumed before playing
Chime: 4-note ascending tone (C5 → E5 → G5 → C6), sine wave, 180ms apart
navigator.vibrate is not used (not supported on iOS)



PWA Configuration


manifest.json: all paths relative (./), display: standalone, theme color #C8401A
sw.js: caches index.html, manifest.json, and both icons on install; serves from cache on fetch
Apple touch icon declared in <head> for iOS home screen icon
apple-mobile-web-app-capable and apple-mobile-web-app-status-bar-style meta tags set
Safe area insets applied via env(safe-area-inset-*) for notch/Dynamic Island devices



Visual Design


Primary color: #C8401A (deep red-orange)
Font: -apple-system / SF Pro (native iOS feel)
Tap targets minimum 44px tall
Active states on all interactive elements
Status bar area filled with primary color
Ingredient rows: tap anywhere on row to toggle checkbox (not just the checkbox itself)
Precook banner: yellow/amber (#FFFBEA, border #E6A817)
Alert banner: light red (#FEF3EE, border #C8401A)
Done banner: green (#EAF3DE)



Known Platform Notes


iOS does not support navigator.vibrate — vibration was removed
Audio must be unlocked via a user gesture before timers can trigger sound — handled via touchstart listener
GitHub Pages serves from a subpath, not root — all asset references must be relative to avoid 404s on PWA launch
After updating files on GitHub, users must delete and re-add the home screen icon to pick up service worker changes
