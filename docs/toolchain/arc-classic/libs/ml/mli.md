# embARC Machine Learning Inference (MLI) Library

![Logo MLI](images/logo-mli.png){ width="200" }

## Overview

The embARC Machine Learning Inference (embARC MLI) library consists of software
functions optimized for DSP-enhanced ARC EMxD and ARC HS4xD processors.
The primary purpose of this library is to enable developers to efficiently
implement and/or port data processing algorithms based on machine learning
principles for ARC Processors. The embARC MLI distribution is managed by
Synopsys for the community and all contributions are welcomed.

The library is a collection of ML algorithms (primitives) that roughly can be
separated into the following groups:

* Convolution – convolve input features with a set of trained weights
* Pooling – pool input features with a function
* Common – Common ML, mathematical, and statistical operations
* Transform – Transform each element of input set according to a particular function
* Element-wise – Apply multi-operand function element-wise to several inputs
* Data manipulation – Move input data by a specified pattern

MLI supported primitives are intended for:

* Ease of use
* Inferring efficient solutions for small/middle models using very limited resources.

## Resources

* [Source Repository](https://github.com/foss-for-synopsys-dwc-arc-processors/embarc_mli)
* [Releases Page](https://github.com/foss-for-synopsys-dwc-arc-processors/embarc_mli)
* [Documentation](https://foss-for-synopsys-dwc-arc-processors.github.io/embarc_mli)
* [Support](https://github.com/foss-for-synopsys-dwc-arc-processors/embarc_mli/issues)
