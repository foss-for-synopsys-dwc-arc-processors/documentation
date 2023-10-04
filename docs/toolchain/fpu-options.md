# Floating Point Unit Options

You can choose a subset of floating point options to be generate for a
particular binary. The subset is chosen by `-mfpu=...` option.
Here is a table which matches `-mfpu=` values to corresponding features for
ARC EM and ARC HS3x/4x as they defined in ARChitect.

| `-mfpu=`    | Families | `-has_fpu` | `-fpu_dp_option` | `-fpu_div_option` | `-fpu_fma_option` | `-fpu_dp_assist` |
|-------------|----------|------------|------------------|-------------------|-------------------|------------------|
| `fpus`      | EM, HS   | On         | —                | —                 | —                 | —                |
| `fpus_div`  | EM, HS   | On         | —                | On                | —                 | —                |
| `fpus_fma`  | EM, HS   | On         | —                | —                 | On                | —                |
| `fpus_all`  | EM, HS   | On         | —                | On                | On                | —                |
| `fpud`      | HS       | On         | On               | —                 | —                 | —                |
| `fpud_div`  | HS       | On         | On               | On                | —                 | —                |
| `fpud_fma`  | HS       | On         | On               | —                 | On                | —                |
| `fpud_all`  | HS       | On         | On               | On                | On                | —                |
| `fpuda`     | EM       | On         | —                | —                 | —                 | On               |
| `fpuda_div` | EM       | On         | —                | On                | —                 | On               |
| `fpuda_fma` | EM       | On         | —                | —                 | On                | On               |
| `fpuda_all` | EM       | On         | —                | On                | On                | On               |