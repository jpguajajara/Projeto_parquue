# Projeto_parque

var fabdem = ee.ImageCollection("projects/sat-io/open-datasets/FABDEM"),
    countries = ee.FeatureCollection("FAO/GAUL/2015/level0"),
    imageVisParam = {"opacity":1,"bands":["b1"],"max":1550.551025390625,"gamma":1},
    geometry = 
    /* color: #98ff00 */
    /* displayProperties: [
      {
        "type": "rectangle"
      }
    ] */
    ee.Geometry.Polygon(
        [[[-49.26823916995046,-16.640819496091407],
          [-49.25373378359792,-16.62502953478893],
          [-49.26823916995046,-16.62502953478893],
          [-49.26823916995046,-16.640819496091407]]], null, false);
print('FABDEM Collection size :',fabdem.size())

//Explanation on setting default Projection here  

 / 1494038930643042309  
var elev = fabdem.mosaic().setDefaultProjection('EPSG:3857',null,30)

//you can also use this incase you don't want to specify CRS
//var elev = fabdem.mosaic().setDefaultProjection(glo30.first().projection())

// Add the elevation to the map.  Play with the visualization tools
// to get a better visualization.
Map.addLayer(elev, {}, 'elev', false);

// Use the terrain algorithms to compute a hillshade with 8-bit values.
var shade = ee.Terrain.hillshade(elev);
Map.addLayer(shade, {}, 'hillshade', false);

// Create an "ocean" variable to be used for cartographic purposes
var ocean = elev.lte(0);
Map.addLayer(ocean.mask(ocean), {palette:'000022'}, 'ocean', false);

// Create a custom elevation palette from hex strings.
var elevationPalette = ['006600', '002200', 'fff700', 'ab7634', 'c4d0ff', 'ffffff'];
// Use these visualization parameters, customized by location.
var visParams = {min: 1, max: 3000, palette: elevationPalette};

// Create a mosaic of the ocean and the elevation data
var visualized = ee.ImageCollection([
  // Mask the elevation to get only land
  elev.mask(ocean.not()).visualize(visParams), 
  // Use the ocean mask directly to display ocean.
  ocean.mask(ocean).visualize({palette:'000022'})
]).mosaic();

// Note that the visualization image doesn't require visualization parameters.
Map.addLayer(visualized.clip(countries), {}, 'elev palette');

// Export the image, specifying scale and region.
// alterar apenas o description
Export.image.toDrive({
  image: elev,
  description: 'Parque Leolideo',
  scale: 30,
  region: geometry
});
