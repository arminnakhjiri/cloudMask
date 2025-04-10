# cloudMaskL457 – Cloud and Shadow Masking for Landsat 4, 5, and 7 in Google Earth Engine

This function masks clouds and cloud shadows from Landsat 4, 5, or 7 imagery using the `pixel_qa` band available in surface reflectance products.

## 🔍 Description

The `cloudMaskL457` function identifies and masks out pixels affected by:

- **Clouds** (bit 5 and bit 7)
- **Cloud shadows** (bit 3)
- **Edge pixels** that don’t appear in all bands

The result is a cleaner image suitable for time-series analysis, classification, or composite generation.

## 🧠 Bitmask Logic

- `bit 5` → Cloud
- `bit 7` → Cloud confidence
- `bit 3` → Cloud shadow

The function masks pixels where:
- Cloud **and** high cloud confidence are detected  
- Cloud shadow is detected  
- Not all bands are valid at the pixel (edge pixels)

## 📦 Function Definition

```javascript
var cloudMaskL457 = function(image) {
  var qa = image.select('pixel_qa');

  // If the cloud bit (5) is set and the cloud confidence (7) is high,
  // or the cloud shadow bit is set (3), then it's a bad pixel.
  var cloud = qa.bitwiseAnd(1 << 5)
                .and(qa.bitwiseAnd(1 << 7))
                .or(qa.bitwiseAnd(1 << 3));

  // Remove edge pixels that don't occur in all bands
  var mask2 = image.mask().reduce(ee.Reducer.min());

  return image.updateMask(cloud.not()).updateMask(mask2);
};
