# Virtual Windchime

A virtual windchime application built with Tauri and SvelteKit. Plays random chime sounds at configurable intervals.

## Features

- Random chime sounds using pentatonic scale for pleasant audio
- Adjustable frequency slider (1-20 seconds average between chimes)
- Start/Stop controls
- Clean, responsive UI with DaisyUI components

## Version 0.2.0 Changes

- Fixed accessibility issue by properly associating label with input control
- Updated to Svelte 5 syntax (using `onclick` instead of `on:click`)
- Added proper AudioContext initialization with user gesture handling
- Implemented cleanup on component destroy to prevent memory leaks
- Added error handling for audio operations
- Improved chime sound with longer decay (2 seconds) and pentatonic scale
- Added visual feedback for audio initialization state
- Fixed AppImage build issue by disabling binary stripping (NO_STRIP=1)

## Developing

Once you've installed dependencies with `npm install`, start a development server:

```sh
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```sh
npm run build
```

You can preview the production build with `npm run preview`.

## Tauri Development

To run the Tauri desktop app in development mode:

```sh
npm run tauri dev
```

To build the Tauri desktop app for production:

```sh
npm run tauri:build
```

Note: The build uses `NO_STRIP=1` to avoid issues with newer binary formats on Arch Linux.

## Technical Details

- **Frontend**: SvelteKit with Svelte 5
- **Styling**: Tailwind CSS with DaisyUI
- **Desktop**: Tauri 2.x
- **Audio**: Web Audio API
