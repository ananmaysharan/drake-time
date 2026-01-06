# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Drake Clock is a single-page web application that displays the current time in different cities with Drake-themed backgrounds and music easter eggs. The application is built with vanilla HTML, CSS, and JavaScript - no build system or package manager required.

## Architecture

### Single-Page Application Structure

This is a purely client-side application with all logic contained in one HTML file:

- **index.html** - Contains all HTML structure, inline JavaScript application logic, and external API integrations
- **styles.css** - All styling including responsive design, glassmorphism effects, and animations
- **drake.png** - Main Drake image displayed on the page

### Key Features & Systems

#### 1. Time Display System
- Shows current time with optional exact time or rounded to nearest hour
- Supports timezone conversion when cities are selected
- Time travel mode (debug mode) allows manually setting the hour via slider
- Uses `Intl.DateTimeFormat` for timezone handling

#### 2. Location & Geolocation
- Auto-detects user location on load via browser geolocation API
- City search powered by GeoNames API (`GEONAMES_USERNAME = 'ananmay01'`)
- Timezone lookup via GeoNames API
- Location determines background, temperature, and easter egg triggers

#### 3. Background System
- Four time-of-day backgrounds in `bgs/`: morning (6-11am), day (12-5pm), evening (6-8pm), night (9pm-5am)
- Crossfade transitions using CSS pseudo-elements and dynamic style injection
- Background changes based on current/selected hour

#### 4. Music Easter Eggs
- Plays Drake songs when specific hour + city + country combinations match
- Easter egg mappings defined in `easterEggSongs` array:
  - 4pm in Calabasas, USA
  - 5am in Toronto, Canada
  - 6pm in New York, USA
  - 7am in Bridle Path, Canada
  - 8am in Charlotte, USA
  - 9am in Dallas, USA
- Audio player with glassmorphic UI appears when easter egg triggered
- Song metadata (title, artist, album art) in `songMetadata` object
- Can be toggled on/off in settings

#### 5. Temperature Display
- Uses Open-Meteo API to fetch hourly temperatures
- Auto-detects unit preference (Fahrenheit for US, Celsius otherwise)
- Manual unit toggle persists to localStorage
- Shows/hides based on settings toggle
- Click temperature display to toggle units

#### 6. Settings System
- Glassmorphic settings panel with toggles:
  - Change Location
  - Exact Time (shows HH:MM instead of rounded hour)
  - Time Travel (debug mode with hour slider)
  - Temperature display
  - Temperature unit (°C/°F pill toggle)
  - Music Easter Eggs
- Settings persisted to `localStorage` as `drakeClockSettings`
- Time travel always resets to off on page load

#### 7. UI/UX Components
- Glassmorphism effects using SVG filters (`#liquid-glass-filter`)
- Blur overlay for modal states (search, settings, credit modal)
- City search with dropdown autocomplete
- Responsive design with mobile breakpoint at 700px
- Custom toggle switches and pill toggles

### External API Dependencies

1. **GeoNames API** - City search and timezone lookup
   - Username: `ananmay01` (hardcoded in script)
   - Used for: `searchJSON`, `timezoneJSON`

2. **BigDataCloud API** - Reverse geocoding (coordinates to city/country)
   - Used for: Initial location detection

3. **Open-Meteo API** - Weather data (temperature)
   - Fetches 24-hour temperature forecast
   - No API key required

4. **Phosphor Icons** - Icon library loaded via CDN
   - Used for: Settings gear, play/pause, info, arrows, music notes

### State Management

Global state variables in inline script:
- `city`, `country` - Current location names
- `selectedTimezone` - IANA timezone string (null = local time)
- `selectedLat`, `selectedLng` - Coordinates for temperature API
- `debugMode`, `debugHour` - Time travel state
- `audioPlayer`, `currentSongPath` - Music playback state
- `hourlyTemperatures` - Array of 24 hourly temps from API
- `settings` - Object persisted to localStorage
- `isSearchMode`, `isSettingsOpen`, `isTimeTravelOpen`, `isCreditModalOpen` - UI state flags

### File Organization

```
/
├── index.html           # Main application file
├── styles.css          # All styles
├── drake.png          # Main character image
├── bgs/               # Time-of-day backgrounds
│   ├── morning.png
│   ├── day.png
│   ├── evening.png
│   └── night.png
├── songs/             # Drake songs for easter eggs
│   ├── Drake - 4pm in Calabasas.mp3
│   ├── Drake - 5 Am in Toronto.mp3
│   ├── Drake - 6PM In New York.mp3
│   ├── Drake - 7am On Bridle Path.mp3
│   ├── Drake - 8am in Charlotte.mp3
│   └── Drake - 9am In Dallas.mp3
├── album-art/         # Album artwork for music player
│   ├── 4pm-in-calabasas.jpg
│   ├── 5am-in-toronto.jpg
│   ├── 6pm-in-new-york.jpg
│   ├── 7am-on-bridle-path.jpg
│   ├── 8am-in-charlotte.jpg
│   └── 9am-in-dallas.jpg
└── assets/
    └── favicon/       # Favicon files
```

## Development

### Running Locally

This is a static site with no build process. Simply open `index.html` in a browser or use any static file server:

```bash
# Using Python
python3 -m http.server 8000

# Using Node.js http-server
npx http-server

# Using PHP
php -S localhost:8000
```

### Testing Easter Eggs

Use the Time Travel feature (enable in settings) to change the hour, then search for specific cities:
- Set hour to 16 (4pm), search "Calabasas" → triggers "4pm in Calabasas"
- Set hour to 5 (5am), search "Toronto" → triggers "5 Am in Toronto"
- etc.

### Modifying Easter Eggs

To add/modify easter eggs, update two arrays in index.html:

1. `easterEggSongs` - Add mapping: `{ hour: X, city: 'name', country: 'name', song: 'path' }`
2. `songMetadata` - Add metadata: `'path': { title: '...', artist: '...', art: '...' }`

City/country matching is case-insensitive and uses `.includes()` for partial matches.

### Styling Notes

- Glassmorphism effects use SVG filter `#liquid-glass-filter` with blur and saturation
- Glass containers have layered structure: `-glass` div for effect, `-content` div for content
- Responsive breakpoint: 700px (mobile adjustments)
- Custom CSS properties used for dynamic background transitions (`--next-bg`)

### State Persistence

Settings are saved to localStorage on every change. To reset:
```javascript
localStorage.removeItem('drakeClockSettings')
```

### API Rate Limits

Be aware of GeoNames API usage limits. The username `ananmay01` is hardcoded - for production use, consider:
- Using environment variables or config
- Implementing request debouncing (already done for city search - 300ms)
- Adding error handling for rate limit responses
