# Multiplication Options

You can choose a subset of multiplication options to be generate for a
particular binary. The subset is chosen by `-mmpy-option=...` option.
Here is a table which matches `-mmpy-option=` values to corresponding features
for ARC EM and ARC HS3x/4x as they defined in ARChitect.

| `-mmpy-option=` | Aliases       | ARChitect option for ARC HS | ARChitect option for ARC EM |
|----------------:|---------------|-----------------------------|-----------------------------|
|               0 | `none`        | `-mpy_option=none`          | `-mpy_option=none`          |
|               2 | `wlh1`, `mpy` | `-mpy_option=mpy`           | `-mpy_option=wlh1`          |
|               3 | `wlh2`        | —                           | `-mpy_option=wlh2`          |
|               4 | `wlh3`        | —                           | `-mpy_option=wlh3`          |
|               5 | `wlh4`        | —                           | `-mpy_option=wlh4`          |
|               6 | `wlh5`        | —                           | `-mpy_option=wlh5`          |
|               7 | `plus_dmpy`   | `-mpy_option=plus_dmpy`     | —                           |
|               8 | `plus_macd`   | `-mpy_option=plus_macd`     | —                           |
|               9 | `plus_qmacw`  | `-mpy_option=plus_qmacw`    | —                           |
