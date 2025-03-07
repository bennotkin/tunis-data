# README

Data for Tunis, Tunisia.

GeoTiffs have been converted to cloud-optimized GeoTiffs using `gdal_translate`. In the future, we will add vector files (likely flatgeobuf), and will use the appropriate parameters for each file (e.g., not use mode as the resampling method and deflate as the compresssion method for all files).

```{r}
source("R/setup.R")

cogs_dir <- file.path(process_output_dir, "spatial-cogs")
dir.create(cogs_dir)

gdal_translate_system <- function(in_file, out_file) {
  # browser()
  system(glue(.sep = " ",
    "gdal_translate {in_file} {out_file} -of COG",
      "-co COMPRESS=DEFLATE",
      "-co PREDICTOR=2",
      "-co RESAMPLING=MODE",
      "-co BLOCKSIZE=512",
      "-co OVERVIEWS=AUTO"))
}

names(layer_params) %>%
  walk(\(layer) {
    # browser()
    print(layer)
    fuzzy_string <- layer_params[[layer]]$fuzzy_string
    if (is.null(fuzzy_string)) return(NULL)
    file <- fuzzy_read(spatial_dir, fuzzy_string, paste)
    if (is.character(file) && str_detect(file, ".tif$")) {
      print("converting")
      cog <- file.path(cogs_dir, basename(file))
      gdal_translate_system(file, cog)
    }
  }
)
```
