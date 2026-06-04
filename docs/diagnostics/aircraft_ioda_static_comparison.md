# Aircraft IODA static comparison

## Files

- Workflow Aircraft link: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/obs/aircraft_obs_2018041500.h5`
- Workflow Aircraft real path: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data/observations/ioda/2018041500/aircraft_obs_2018041500.h5`
- Official MPAS-JEDI Aircraft source: `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/aircraft_obs_2018041500_m.nc4`
- Official MPAS-JEDI Aircraft build copy: `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/Data/ufo/testinput_tier_1/aircraft_obs_2018041500_m.nc4`

| file | size bytes | root groups | nlocs |
|---|---:|---|---:|
| workflow | 57243706 | `Location, MetaData, ObsError, ObsType, ObsValue, PreQC, nstring, nvars` | 348901 |
| official | 343800 | `GsiAdjustObsError, GsiFinalObsError, GsiHofX, GsiHofXBc, GsiQCWeight, GsiUseFlag, Location, MetaData, ObsError, ObsType, ObsValue, PreQC, PreUseFlag` | 2103 |

## Root groups and IODA groups

### workflow

- `MetaData`: `dateTime, height, latitude, longitude, pressure, stationElevation, stationIdentification, variable_names`
- `ObsValue`: `airTemperature, specificHumidity, virtualTemperature, windEastward, windNorthward`
- `ObsError`: `airTemperature, specificHumidity, virtualTemperature, windEastward, windNorthward`
- `PreQC`: `airTemperature, specificHumidity, virtualTemperature, windEastward, windNorthward`
- `GsiHofXBc`: absent
- `GsiHofX`: absent
- `GsiFinalObsError`: absent
- `GsiEffectiveQC`: absent
- `GsiAdjustObsError`: absent
- `EffectiveQC`: absent
- `EffectiveError`: absent

### official

- `MetaData`: `dateTime, height, latitude, longitude, pressure, sequenceNumber, stationElevation, stationIdentification`
- `ObsValue`: `airTemperature, specificHumidity, windEastward, windNorthward`
- `ObsError`: `airTemperature, specificHumidity, windEastward, windNorthward`
- `PreQC`: `airTemperature, specificHumidity, windEastward, windNorthward`
- `GsiHofXBc`: `airTemperature, specificHumidity, windEastward, windNorthward`
- `GsiHofX`: `airTemperature, specificHumidity, windEastward, windNorthward`
- `GsiFinalObsError`: `airTemperature, specificHumidity, windEastward, windNorthward`
- `GsiEffectiveQC`: absent
- `GsiAdjustObsError`: `airTemperature, specificHumidity, windEastward, windNorthward`
- `EffectiveQC`: absent
- `EffectiveError`: absent

## ObsValue variables

- workflow: `airTemperature, specificHumidity, virtualTemperature, windEastward, windNorthward`
- official: `airTemperature, specificHumidity, windEastward, windNorthward`

## ObsError variables

- workflow: `airTemperature, specificHumidity, virtualTemperature, windEastward, windNorthward`
- official: `airTemperature, specificHumidity, windEastward, windNorthward`

## PreQC variables

- workflow: `airTemperature, specificHumidity, virtualTemperature, windEastward, windNorthward`
- official: `airTemperature, specificHumidity, windEastward, windNorthward`

## Essential MetaData

| variable | workflow | official |
|---|---|---|
| `latitude` | shape=(348901,); dtype=float32; range=-71.0999984741211..83.2300033569336; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=-35.20000076293945..64.927001953125; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['degrees_north']} |
| `longitude` | shape=(348901,); dtype=float32; range=-179.9980010986328..180.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=103.65833282470703..289.4200134277344; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['degrees_east']} |
| `dateTime` | shape=(348901,); dtype=int64; range=1523739600.0..1523761200.0; utc=2018-04-14T21:00:00Z..2018-04-15T03:00:00Z; attrs={'_Netcdf4Coordinates': [1], 'units': 'seconds since 1970-01-01T00:00:00Z', 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=int64; range=1523739600.0..1523761200.0; utc=2018-04-14T21:00:00Z..2018-04-15T03:00:00Z; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [-2208988800], 'units': ['seconds since 1970-01-01T00:00:00Z']} |
| `pressure` | shape=(348901,); dtype=float32; range=14750.0..103630.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=17870.0..102160.0; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['Pa']} |
| `height` | shape=(348901,); dtype=float32; range=-190.0..13716.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=-69.0..12500.0; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['m']} |
| `stationIdentification` | shape=(348901,); dtype=object; sample=['0QCUN', '0QCUN', '0QCUN', '0QCUN', '0QCUN']; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [''], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=object; sample=['00001070', '00001070', '00001070', '00001070', '00001070']; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': ['']} |
| `recordNumber` | absent | absent |
| `aircraftIdentifier` | absent | absent |
| `sensorCentralFrequency` | absent | absent |
| `sensorChannelNumber` | absent | absent |

## ObsValue dataset details

| dataset | workflow | official |
|---|---|---|
| `airTemperature` | shape=(348901,); dtype=float32; range=-999.0..353.25; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'units': 'K', 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=215.14999389648438..9.969209968386869e+36; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['K']} |
| `specificHumidity` | shape=(348901,); dtype=float32; range=-999.0..0.023838000372052193; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'units': 'kg/kg', 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=1.9999999949504854e-06..9.969209968386869e+36; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['1']} |
| `virtualTemperature` | shape=(348901,); dtype=float32; range=-999.0..-999.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'units': 'K', 'DIMENSION_LIST': [[<HDF5 object reference>]]} | absent |
| `windEastward` | shape=(348901,); dtype=float32; range=-999.0..92.9000015258789; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'units': 'm/s', 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=-12.699999809265137..9.969209968386869e+36; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['m s-1']} |
| `windNorthward` | shape=(348901,); dtype=float32; range=-999.0..114.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'units': 'm/s', 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=-52.900001525878906..9.969209968386869e+36; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['m s-1']} |

## ObsError dataset details

| dataset | workflow | official |
|---|---|---|
| `airTemperature` | shape=(348901,); dtype=float32; range=-999.0..3.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'units': 'K', 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=1.0..9.969209968386869e+36; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['K']} |
| `specificHumidity` | shape=(348901,); dtype=float32; range=-999.0..0.012749646790325642; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'units': 'kg/kg', 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=6.34090702078538e-06..9.969209968386869e+36; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['1']} |
| `virtualTemperature` | shape=(348901,); dtype=float32; range=-999.0..-999.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'units': 'K', 'DIMENSION_LIST': [[<HDF5 object reference>]]} | absent |
| `windEastward` | shape=(348901,); dtype=float32; range=-999.0..6.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'units': 'm/s', 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=2.5..9.969209968386869e+36; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['m s-1']} |
| `windNorthward` | shape=(348901,); dtype=float32; range=-999.0..6.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'units': 'm/s', 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=2.5..9.969209968386869e+36; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['m s-1']} |

## PreQC dataset details

| dataset | workflow | official |
|---|---|---|
| `airTemperature` | shape=(348901,); dtype=int32; range=-999.0..114.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=int32; range=-2147483647.0..14.0; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [-2147483647]} |
| `specificHumidity` | shape=(348901,); dtype=int32; range=-999.0..115.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=int32; range=-2147483647.0..15.0; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [-2147483647]} |
| `virtualTemperature` | shape=(348901,); dtype=int32; range=-999.0..-999.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | absent |
| `windEastward` | shape=(348901,); dtype=int32; range=-999.0..114.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=int32; range=-2147483647.0..13.0; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [-2147483647]} |
| `windNorthward` | shape=(348901,); dtype=int32; range=-999.0..114.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=int32; range=-2147483647.0..13.0; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [-2147483647]} |

## MetaData dataset details

| dataset | workflow | official |
|---|---|---|
| `dateTime` | shape=(348901,); dtype=int64; range=1523739600.0..1523761200.0; attrs={'_Netcdf4Coordinates': [1], 'units': 'seconds since 1970-01-01T00:00:00Z', 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=int64; range=1523739600.0..1523761200.0; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [-2208988800], 'units': ['seconds since 1970-01-01T00:00:00Z']} |
| `height` | shape=(348901,); dtype=float32; range=-190.0..13716.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=-69.0..12500.0; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['m']} |
| `latitude` | shape=(348901,); dtype=float32; range=-71.0999984741211..83.2300033569336; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=-35.20000076293945..64.927001953125; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['degrees_north']} |
| `longitude` | shape=(348901,); dtype=float32; range=-179.9980010986328..180.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=103.65833282470703..289.4200134277344; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['degrees_east']} |
| `pressure` | shape=(348901,); dtype=float32; range=14750.0..103630.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=17870.0..102160.0; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['Pa']} |
| `sequenceNumber` | absent | shape=(2103,); dtype=int32; range=65.0..3446.0; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [-2147483647]} |
| `stationElevation` | shape=(348901,); dtype=float32; range=-190.0..13716.0; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [-999.0], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=float32; range=-50.0..12500.0; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': [9.969209968386869e+36], 'units': ['m']} |
| `stationIdentification` | shape=(348901,); dtype=object; attrs={'_Netcdf4Coordinates': [1], '_FillValue': [''], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | shape=(2103,); dtype=object; attrs={'DIMENSION_LIST': [[<HDF5 object reference>]], '_FillValue': ['']} |
| `variable_names` | shape=(5,); dtype=object; attrs={'_Netcdf4Coordinates': [0], '_FillValue': [''], 'DIMENSION_LIST': [[<HDF5 object reference>]]} | absent |

## Material differences

- `ObsValue` differs: only workflow=['virtualTemperature']; only official=[].
- `ObsError` differs: only workflow=['virtualTemperature']; only official=[].
- `PreQC` differs: only workflow=['virtualTemperature']; only official=[].
- `MetaData` differs: only workflow=['variable_names']; only official=['sequenceNumber'].
- Number of locations differs: workflow=348901, official=2103.
- `dateTime` shape/dtype differs: workflow shape=(348901,) dtype=int64; official shape=(2103,) dtype=int64.
- `pressure` shape/dtype differs: workflow shape=(348901,) dtype=float32; official shape=(2103,) dtype=float32.
- `height` shape/dtype differs: workflow shape=(348901,) dtype=float32; official shape=(2103,) dtype=float32.
- `latitude` shape/dtype differs: workflow shape=(348901,) dtype=float32; official shape=(2103,) dtype=float32.
- `longitude` shape/dtype differs: workflow shape=(348901,) dtype=float32; official shape=(2103,) dtype=float32.

## Diagnostic interpretation

- The workflow file is the Aircraft input used by the isolated failing YAML; it is a symlink to the prepared runtime.
- The official file is the Aircraft IODA referenced by MPAS-JEDI `test/testinput/3dvar.yaml`, which has already passed via CTest in this environment.
- Differences above that affect `MetaData/pressure`, `MetaData/dateTime`, location count, variable naming, or GSI auxiliary groups are the highest-priority candidates for a `VertInterp`/HofX crash.
- This report is static only; no MPAS-JEDI executable was run during the comparison.
