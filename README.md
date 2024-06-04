# Vancouver Public Art Map

*Last updated September 2023*

Interactive visualization of public artworks in the City of Vancouver boundary. When a user clicks on a point on the map, a popup window will appear on the left. This popup will present details about the selected artwork: artwork name, an image, art type, a concise description, current status (whether the artwork is still in place or has been removed/deaccessioned), primary materials used, location address, and the year of installation.

## Data Sources:
- City of Vancouver [Local area boundary] (https://opendata.vancouver.ca/explore/dataset/local-area-boundary/information/?disjunctive.name&location=11,49.26825,-123.02525) (Last modified: June 24, 2023)
- City of Vancouver [Public art] (https://opendata.vancouver.ca/explore/dataset/public-art/information/) (Last modified: August 14, 2023)

### Technology used:
MapLibre GL JS, Maputnik, Stadia Maps, svelte, QGIS


### Developing Locally

Run the following your terminal

```
git clone https://github.com/schoolofcities/vancouver-public-art
cd vancouver-public-art
npm install
npm run dev
```

Then go to `http://localhost:5173/vancouver-public-art/map`

To build...

```
npm run build
```