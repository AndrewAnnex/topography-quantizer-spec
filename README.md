# topography-quantizer-spec
Specification for topography (elevation) quantizers useful for enoding floating point elevation data as 24bit RGB terrain tiles.

Terrain tiles are created using a quantizer that is defined by two parameters, a resolution in meters and a minimum elevation in meters. 
Elevations >= the minimum elevation can be encoded up to 16,777,216 times the resolution. 

Given a spec, terrain can be encoded to 24 bits using the following function:
```python
def encode_to_rgb(z: float, min_elevation: float, resolution: float):
    z -= min_elevation
    z /= resolution
    d_z = z / 256
    dd_z = z // 256
    ddd_z = dd_z // 256
    b = (d_z - dd_z) * 256
    g = ((dd_z / 256) - ddd_z) * 256
    r = ((ddd_z / 256) - (ddd_z // 256)) * 256
    return np.uint8(r), np.uint8(g), np.uint(b)
```

and converted back to floating point by:
```python
def to_z(r: int, g:int, b:int, min_elevation: float, resolution: float):
    return min_elevation + ((r << 16) + (g << 8) + b) * resolution
```

Optionally the quantizer can decoder can be defined by three optional scalers which is the product of 256^2, 256, and 1 each multiplied by the resolution for the red, green, and blue bands respectively. Note the blue channel scaler (bScaler) is the same as the resolution.

```python
def to_z(r: int, g:int, b:int, rScaler: float, gScaler: float, bScaler: float, min_elevation: float):
    return min_elevation + (r * rScaler + g * gScaler + b * bScaler)
```


## examples 

Below are several example quantizers for the four inner rocky planets and the earth's moon. The "Anyrock" quantizer is able to represent all topography in the inner solarsystem at a slightly reduced resolution, however approximately 2mm vertical resolution is incredibly high precision for most datasets derived from orbiting spacecraft or in atmosphere. 

```json
    {
      "body": "anyrock",
      "min": -12000.0,
      "resolution": 0.00204682,
      "rScaler": 134.14039552,
      "gScaler": 0.52398592,
      "bScaler": 0.00204682
    },
    {
      "body": "mercury",
      "min": -6000.0,
      "resolution": 0.00066221,
      "rScaler": 43.39859456,
      "gScaler": 0.16952576,
      "bScaler": 0.00066221
    },
    {
      "body": "venus",
      "min": -4000.0,
      "resolution": 0.00102341,
      "rScaler": 67.07019776,
      "gScaler": 0.26199296,
      "bScaler": 0.00102341
    },
    {
      "body": "earth",
      "min": -11500.0,
      "resolution": 0.001264215,
      "rScaler": 82.85126656,
      "gScaler": 0.32363776,
      "bScaler": 0.00126421
    },
    {
      "body": "moon",
      "min": -9500.0,
      "resolution": 0.001264215,
      "rScaler": 82.85126656,
      "gScaler": 0.32363776,
      "bScaler": 0.00126421
    },
    {
      "body": "mars",
      "min": -8500.0,
      "resolution": 0.00178993,
      "rScaler": 117.30485248,
      "gScaler": 0.45822208,
      "bScaler": 0.00178993
    }

```
