# Vue OpenStreetMap Multi-Source Viewer

A Vue 3 application for displaying multiple data sources on an interactive OpenStreetMap using Leaflet. This application allows users to visualize different types of location data (restaurants, parks, etc.) with marker clustering for better performance and user experience.

## Features

- **Multiple Data Sources**: Toggle between different data sources (restaurants, parks, etc.)
- **Interactive Map**: Powered by Leaflet and OpenStreetMap tiles
- **Marker Clustering**: Automatic grouping of nearby markers using leaflet.markercluster
- **Dynamic Loading**: Data is loaded on-demand when sources are selected
- **Color-Coded Markers**: Different colors for different data sources
- **Responsive Design**: Works on desktop and mobile devices

## Technologies

- **Vue 3** - Progressive JavaScript framework with Composition API
- **Vite** - Fast build tool and dev server
- **Leaflet** - Open-source JavaScript library for interactive maps
- **Leaflet.markercluster** - Plugin for marker clustering

## Installation

```bash
# Install dependencies
npm install
```

## Usage

### Development Server

```bash
npm run dev
```

Visit `http://localhost:5173` (or the port shown in terminal)

### Build for Production

```bash
npm run build
```

### Preview Production Build

```bash
npm run preview
```

## Project Structure

```
src/
├── components/
│   ├── MapView.vue          # Main map component with Leaflet integration
│   ├── SourceSelector.vue   # UI for selecting data sources
│   └── Sidebar.vue          # Sidebar for displaying marker details
├── App.vue                  # Root component
└── main.js                  # Application entry point

public/
└── data/                    # JSON files for different data sources
    ├── restaurants.json
    └── parks.json
```

## Adding New Data Sources

1. Create a JSON file in `public/data/` with the following structure:

```json
[
  {
    "id": 1,
    "name": "Location Name",
    "lat": 40.7128,
    "lng": -74.0060,
    "description": "Optional description",
    "address": "Optional address"
  }
]
```

2. Add the source to the `sources` array in `src/App.vue`:

```javascript
const sources = [
  {
    id: 'source-id',
    name: 'Display Name',
    url: '/data/filename.json',
    color: '#hexcolor',
    selectedColor: '#hexcolor'
  }
]
```

## License

MIT
