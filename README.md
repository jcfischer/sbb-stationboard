# SBB Swiss Stationboard for TRMNL

A TRMNL e-ink display plugin showing live departure information for Swiss public transport stations (trains, trams, buses, ships, cable cars).

## Features

- **Live departure information** from any Swiss public transport station
- **Multiple layout variants** - optimized for full screen, half screen (horizontal/vertical), and quadrant displays
- **Smart filtering** - hide specific transport types, lines, or destinations
- **Walking time calculation** - shows when you need to leave to catch your connection
- **Delay tracking** - displays real-time delay information
- **Customizable display** - abbreviate destination names, hide words, filter by transport type
- **Icon-based transport types** - visual indicators for trains, trams, buses, ships, and cable cars

## Installation

### Prerequisites

You need the **TRMNL Developer edition** to use private plugins:
- Upgrade via the device picker dropdown → gear icon → "Developer perks"
- One-time fee of $20 (or included with Developer edition purchase)
- Free for BYOD (Bring Your Own Device) licenses

### Import the Plugin

1. Download the latest release ZIP file from this repository
2. Go to your [TRMNL Private Plugin settings page](https://usetrmnl.com/plugin_settings?keyname=private_plugin)
3. Click **"Import new"** and select the ZIP file
4. The plugin will be automatically added to your playlist

### Alternative: Manual Setup

If you prefer to set it up manually:

1. Create a new Private Plugin in TRMNL
2. Copy the contents of `settings.yml` to your plugin settings
3. Copy each layout file (`full.liquid`, `half_horizontal.liquid`, etc.) to the corresponding layout in TRMNL
4. Copy `shared.liquid` content (TRMNL will automatically prepend this to each layout)

## Configuration

### Find Your Station

Station IDs must be looked up via the Swiss OpenData Transport API:

```
https://transport.opendata.ch/v1/locations?query=Basel
```

**Common station IDs:**
- Zürich HB: `8503000`
- Bern: `8507000`
- Basel SBB: `8500010`
- Genève: `8501008`
- Lausanne: `8501120`

### Configuration Options

| Field | Description | Example |
|-------|-------------|---------|
| **Station Name** | Display name for your station | `Zürich HB` |
| **Station ID** | OpenData Transport station ID | `8503000` |
| **Walking Time** | Minutes to reach the station | `5` |
| **Min Departure Offset** | Hide departures less than N minutes away | `10` |
| **Hide Transport Types** | Filter by category or type | `bus, tram, IC` |
| **Filter Lines** | Hide specific line+destination combos | `11Auz, 14Tri` |
| **Hide Words** | Remove words from destination names | `Zürich, Bahnhof` |
| **Abbreviations** | Shorten destination names | `Bahnhof=Bhf.` |
| **Cableway Icon** | Preferred icon for cable cars | `Gondel` or `Zahnradbahn` |

### Filter Examples

**Hide all buses and trams:**
```
bus, tram
```

**Hide specific lines:**
- `11Auz` - Line 11 to Auzelg
- `14Tri` - Line 14 to Triemli
- Format: `{lineNumber}{first3CharsOfDestination}`

**Abbreviate destinations:**
```
Bahnhof=Bhf., Zürich=ZH, Hauptbahnhof=HB
```

## Data Source

This plugin uses the [Swiss OpenData Transport API](https://transport.opendata.ch):
- Real-time departure data
- Delay information
- Free, public API
- No authentication required

**API Optimization Tip:** To reduce payload size, add field filters to the polling URL in `settings.yml`:

```yaml
polling_url: https://transport.opendata.ch/v1/stationboard?id={{station_id}}&limit=10&fields[]=stationboard/category&fields[]=stationboard/number&fields[]=stationboard/to&fields[]=stationboard/stop/departure&fields[]=stationboard/stop/prognosis/departure
```

## Architecture

### Files

- **`shared.liquid`** - Common code automatically prepended by TRMNL
  - Helper functions (data parsing, validation)
  - Icon and category mapping
  - Text processing utilities
  - Main rendering function
  - Common CSS utilities

- **Layout files** - Screen size specific templates
  - `full.liquid` - Full screen (12 departures, big labels)
  - `half_horizontal.liquid` - Half screen horizontal (5 departures)
  - `half_vertical.liquid` - Half screen vertical (12 departures)
  - `quadrant.liquid` - Quarter screen (5 departures)

- **`settings.yml`** - Plugin configuration and custom fields

### Benefits of Refactored Architecture

- **Single source of truth** - Business logic in one place
- **No code duplication** - ~200 lines of duplicate code eliminated per layout
- **Easy maintenance** - Fix bugs once, applies to all layouts
- **Clean layouts** - Each layout focuses on structure and styling
- **Extensible** - New layouts can easily include shared functionality

## Development

### Making Changes

**For common functionality:**
- Edit `shared.liquid`
- Changes automatically apply to all layouts

**For layout-specific changes:**
- Edit the specific layout file
- Adjust column widths, maxRows, or labelSize

### Adding Features

1. Add helper functions to `shared.liquid`
2. Update `renderDepartures()` if needed
3. Test with all layout variants
4. Update `settings.yml` for new configuration options

### Testing

Deploy to TRMNL and test with different:
- Station types (urban vs. long-distance)
- Filter combinations
- Screen layouts
- Transport type mixes

## Credits

- **Original Author:** David M. ([Instagram](https://www.instagram.com/david_meury/), [GitHub](https://github.com/notaprogrammer-alt))
- **Updated by:** Jens-Christian Fischer - Refactored for polling, code optimization
- **Data Source:** [Swiss OpenData Transport](https://transport.opendata.ch)
- **Icons:** [Icons8](https://icons8.com)

## Resources

- [TRMNL Private Plugins Documentation](https://help.usetrmnl.com/en/articles/9510536-private-plugins)
- [TRMNL Screen Templating](https://docs.usetrmnl.com/go/private-plugins/templates)
- [Swiss OpenData Transport API Docs](https://transport.opendata.ch/docs.html)
- [Importing/Exporting Plugins](https://help.usetrmnl.com/en/articles/10542599-importing-and-exporting-private-plugins)

## License

This plugin is provided as-is for use with TRMNL devices. Feel free to modify and adapt for your needs.

## Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Test with multiple layout variants
4. Submit a pull request

---

**Note:** This is a private plugin for TRMNL e-ink displays. You need a TRMNL device and Developer edition to use it.
