# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **TRMNL plugin** that displays live Swiss public transport departure information (trains, trams, buses, ships, cable cars) on e-ink displays. The plugin polls the Swiss OpenData Transport API and renders stationboard data using Liquid templates with embedded JavaScript and CSS.

**Key characteristics:**
- Template-based architecture (Liquid templating language)
- Client-side JavaScript for data processing and rendering
- No build system - templates are deployed directly to TRMNL
- Four layout variants for different screen configurations

## Architecture

### Core Components

**settings.yml** - Plugin configuration and metadata
- Defines custom fields (station selection, filters, display options)
- Configures polling strategy (GET request to transport.opendata.ch)
- Sets refresh interval (15 seconds)
- Contains plugin metadata and help text

**Layout Templates** (*.liquid files)
- `full.liquid` - Full screen layout (12 departures, larger text)
- `half_horizontal.liquid` - Half screen horizontal layout
- `half_vertical.liquid` - Half screen vertical layout
- `quadrant.liquid` - Quarter screen layout (5 departures, compact)

### Template Structure

Each layout file contains three sections:

1. **Liquid variables** (lines 1-10)
   - Extract user settings from `trmnl.plugin_settings.custom_fields_values`
   - Examples: `station_name`, `station_id`, `walking_time`, filter configurations

2. **HTML + CSS** (lines 11-132)
   - Responsive table layout with fixed column widths
   - Transport type icons (via icons8.com)
   - Accessibility features (screen reader support)
   - Layout-specific styling (full vs quadrant have different sizes)

3. **JavaScript logic** (lines 134-347)
   - Data processing functions (filtering, abbreviations, hide words)
   - Transport category mapping (IC/IR/S → train, T → tram, etc.)
   - Time calculations (departure time, walking time offset, delay handling)
   - Dynamic table rendering

### Data Flow

1. TRMNL polls `https://transport.opendata.ch/v1/stationboard?id={{station_id}}&limit=10`
2. API returns JSON with stationboard data
3. Liquid injects data into template: `{{ stationboard | json }}`
4. JavaScript processes entries (filtering, sorting, formatting)
5. HTML table is dynamically rendered in `#departures-body`

## Key Features

**Transport Filtering**
- Hide entire transport categories (train, tram, bus, ship, cableway)
- Filter specific lines by line number + destination prefix
- Category mapping: "IC", "IR", "S" → train group, "T" → tram, etc.

**Display Customization**
- Hide words in destination names (e.g., remove "Zürich" or "Bahnhof")
- Abbreviate destination words (e.g., "Bahnhof=Bhf.")
- Configurable cableway icon (Gondola vs Funicular)

**Time Handling**
- Walking time offset: calculates "leave by" time (Abfahrt minus walking time)
- Minimum departure offset: hide departures less than N minutes away
- Delay display: shows +/- minutes from scheduled time
- Swiss timezone (Europe/Zurich) for all time rendering

**Layout Variants**
- **full.liquid**: 12 rows, larger text (`label--big`, `title--big`)
- **quadrant.liquid**: 5 rows, compact text (`label--small`, `title--small`), tighter columns

## Configuration Reference

### Station Configuration

Station IDs must be looked up manually via the Swiss OpenData Transport API:
```
https://transport.opendata.ch/v1/locations?query=Basel
```

Common station IDs:
- Zürich HB: `8503000`
- Zürich, Im Hagacker: `8591209`

### Filter Formats

**transport_hidegroups** (comma-separated)
- Categories: `train`, `tram`, `bus`, `ship`, `cableway`
- Specific types: `IC`, `IR`, `S`, `RE`, `ICE`, `EXT`, `bus`, `tram`

**linetarget_filter** (comma-separated)
- Format: `{lineNumber}{first3CharsOfDestination}`
- Example: `11Auz` filters line 11 to Auzelg, `14Tri` filters line 14 to Triemli

**target_hidewords** (comma-separated)
- Words to remove from destination names
- Example: `Zürich, Bahnhof, City`
- Safety: won't remove if part of "X Y Bahnhof" pattern

**target_abbreviations** (comma-separated key=value pairs)
- Format: `FullWord=Abbr`
- Example: `Bahnhof=Bhf., Stauffacher=Stauff.`

## Development Workflow

### Making Changes

1. Edit the relevant `.liquid` file directly
2. Test changes by deploying to TRMNL (no local testing environment)
3. Changes take effect on next polling cycle (15 seconds)

### Adding New Features

When modifying the JavaScript:
- All utility functions are inline (no imports/modules)
- Helper functions: `isBlank()`, `getData()`, `parseCsv()`, `esc()`, `toInt()`
- Category mapping: `mapCategoryToGroup()` converts types to groups
- Icon rendering: `getLineSymbol()` maps categories to icons8.com URLs

### Layout Adjustments

Column widths are defined in CSS variables:
- Full screen: `col-linie: 39%`, `col-abfahrt: 15%`, `col-delay: 20%`
- Quadrant: `col-linie: 25%`, `col-abfahrt: 15%`, `col-delay: 20%`

Text sizes controlled by class:
- Full: `label--big`, `title--big`
- Quadrant: `label--small`, `title--small`

### Common Tasks

**Change max displayed departures:**
- Edit `const maxRows = 12;` (full.liquid) or `const maxRows = 5;` (quadrant.liquid)

**Add new transport icon:**
- Add category mapping in `mapCategoryToGroup()`
- Add icon URL in `getLineSymbol()`
- Use icons8.com for consistency

**Modify time format:**
- Edit `timeOptions` object (currently: `'de-CH'`, 24-hour format)

**Adjust column spacing:**
- Modify `--gap-45` CSS variable (controls space between Abfahrt and Delay columns)

## API Reference

**Swiss OpenData Transport API**
- Base URL: `https://transport.opendata.ch/v1/`
- Endpoint: `stationboard?id={station_id}&limit={count}`
- Response structure:
  ```
  {
    stationboard: [
      {
        category: "IC",    // Transport type
        number: "1",       // Line number
        to: "Basel SBB",   // Destination
        stop: {
          departure: "2025-11-28T10:30:00+0100",  // ISO timestamp
          prognosis: {
            departure: "2025-11-28T10:33:00+0100"  // Actual time (with delay)
          }
        }
      }
    ]
  }
  ```

## Notes

- No package.json, no dependencies, no build process
- All code runs in TRMNL's sandboxed environment
- Liquid variables are server-side rendered, JavaScript is client-side
- Image assets use base64 data URIs or external icons8.com URLs
- All text is in German (Swiss German conventions)
