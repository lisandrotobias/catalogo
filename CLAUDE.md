# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev`: Start development server (with hot reload)
- `npm run dev -- --open`: Start development server and open in browser
- `npm run build`: Create production build
- `npm run preview`: Preview production build locally
- `npm run prepare`: Sync SvelteKit configuration

## Project Architecture

This is a SvelteKit application with Tailwind CSS that creates an interactive digital catalog/flipbook viewer. The application uses the PageFlip.js library to provide a realistic page-turning experience.

### Key Components

- **Main Application**: Single-page app displaying a digital catalog
- **Catalog Component** (`src/lib/components/Catalog.svelte`): Core flipbook implementation
- **PageFlip Integration**: Uses external PageFlip.js library loaded from CDN
- **Responsive Design**: Adapts layout between mobile (single page) and desktop (spread view)

### Technology Stack

- **Framework**: SvelteKit 5.x
- **Styling**: Tailwind CSS v4
- **Flipbook**: PageFlip.js v2.0.7 (loaded from CDN)
- **Build Tool**: Vite
- **Icons**: FontAwesome (CDN)

### Project Structure

- `src/routes/+page.svelte`: Main page importing the Catalog component
- `src/lib/components/Catalog.svelte`: Interactive flipbook component
- `static/images/trabajo_pou/`: Contains PNG images (1.png through 16.png) displayed as pages
- A4 format optimization with responsive mobile/desktop configurations

### Important Implementation Details

- **Responsive Behavior**: Automatically detects screen size and configures PageFlip for mobile (portrait, single page) vs desktop (landscape, spread view)
- **A4 Aspect Ratio**: Maintains 210:297 ratio for document pages
- **Dynamic Loading**: PageFlip script is loaded dynamically from CDN in onMount
- **Event Handling**: Includes page flip events and window resize handling
- **Mobile Support**: Special configuration for mobile devices with touch support

The application is designed specifically for displaying a 16-page document with navigation controls and automatic responsive behavior.