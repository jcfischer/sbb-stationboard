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

**shared.liquid** - Common code shared across all layouts
- JavaScript helper functions (isBlank, getData, parseCsv, esc, etc.)
- Icon and category mapping functions (mapCategoryToGroup, getLineSymbol, getCablewayIconHtml)
- Text processing utilities (cleanTarget, formatLineNumber, getTargetPrefix)
- Main rendering function (renderDepartures) - parameterized for maxRows and labelSize
- Common CSS utilities (sr-only, time, table base styles, column alignment)

**Layout Templates** (*.liquid files)
- `full.liquid` - Full screen layout (12 departures, big labels)
- `half_horizontal.liquid` - Half screen horizontal layout (5 departures, big labels)
- `half_vertical.liquid` - Half screen vertical layout (12 departures, big labels)
- `quadrant.liquid` - Quarter screen layout (5 departures, big labels)

### Template Structure

Each layout file contains:

1. **Liquid variables** (lines 1-9)
   - Extract user settings from `trmnl.plugin_settings.custom_fields_values`
   - Examples: `station_name`, `station_id`, `walking_time`, filter configurations

2. **HTML structure** (lines 11-48)
   - Flexbox layout with table for departures
   - Transport type icon column, line number, destination, departure time, delay, "leave by" time
   - Title bar with station name and timestamp
   - Accessibility features (screen reader support with .sr-only)

3. **Layout-specific CSS** (lines 50-62)
   - Column widths (different for each layout variant)
   - CSS variable --gap-45 (spacing between columns)
   - Text alignment and padding

4. **Shared code inclusion** (line 64)
   - `{% include 'shared' %}` - Pulls in all common JavaScript and CSS

5. **Layout-specific JavaScript** (lines 66-84)
   - Set data attributes from Liquid variables
   - Inject stationboard data into window.stationboardData
   - Call renderDepartures(maxRows, labelSize) with layout-specific parameters

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

1. **For common functionality changes**: Edit `shared.liquid`
   - Helper functions, icon mapping, text processing, rendering logic
   - Changes automatically apply to all 4 layouts

2. **For layout-specific changes**: Edit the specific layout file
   - HTML structure, column widths, maxRows, labelSize parameters
   - Only affects that particular layout variant

3. Test changes by deploying to TRMNL (no local testing environment)
4. Changes take effect on next polling cycle (15 seconds)

### Adding New Features

When modifying the JavaScript:
- **Common functions go in shared.liquid** (helper functions, icon mapping, etc.)
- **Layout-specific code goes in layout files** (data attributes, renderDepartures call)
- Helper functions: `isBlank()`, `getData()`, `parseCsv()`, `esc()`, `toInt()`
- Category mapping: `mapCategoryToGroup()` converts types to groups
- Icon rendering: `getLineSymbol()` maps categories to icons8.com URLs
- Main rendering: `renderDepartures(maxRows, labelSize)` - called from each layout

### Layout Adjustments

Column widths are defined in CSS variables:
- Full screen: `col-linie: 39%`, `col-abfahrt: 15%`, `col-delay: 20%`
- Quadrant: `col-linie: 25%`, `col-abfahrt: 15%`, `col-delay: 20%`

Text sizes controlled by class:
- Full: `label--big`, `title--big`
- Quadrant: `label--small`, `title--small`

### Common Tasks

**Change max displayed departures:**
- Edit the `renderDepartures()` call in the specific layout file
- Example: Change `renderDepartures(12, 'big')` to `renderDepartures(15, 'big')`

**Add new transport icon:**
- Edit `shared.liquid`:
  - Add category mapping in `mapCategoryToGroup()`
  - Add icon URL in `getLineSymbol()`
  - Use icons8.com for consistency
- Changes apply to all layouts automatically

**Modify time format:**
- Edit `shared.liquid`:
  - Update `timeOptions` object in `renderDepartures()` function
  - Currently: `'de-CH'`, 24-hour format

**Adjust column spacing:**
- Edit the specific layout file:
  - Modify `--gap-45` CSS variable (controls space between Abfahrt and Delay columns)
  - Different layouts can have different spacing

**Add new helper function:**
- Edit `shared.liquid`:
  - Add function in the appropriate section (Helper Functions, Text Processing, etc.)
  - Function becomes available to all layouts

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

## Architecture Changes

**Refactored (Current State):**
- Common code extracted to `shared.liquid` (helper functions, icon mapping, rendering logic, common CSS)
- Layout files include shared code via `{% include 'shared' %}`
- Each layout calls `renderDepartures(maxRows, labelSize)` with specific parameters
- Eliminates ~200+ lines of duplicate code per file
- Single source of truth for business logic - changes apply to all layouts

**Benefits:**
- Much easier to maintain - fix bugs once instead of in 4 places
- Safer refactoring - no risk of layouts diverging
- Cleaner layout files - focus on structure and styling, not logic
- New layouts can be added easily by including shared code
