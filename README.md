# CURAND.jl

Wrapper for NVidia's cuRAND library. 

## High-level interface

CURAND.jl provides the following list of high-level functions: 

 * `curand` - generate CudaArray of uniformly distributed numbers
 * `curandn` - generate CudaArray of normally distributed numbers
 * `curand_logn` - generate CudaArray of log-normally distributed numbers
 * `curand_poisson` - generate numbers according to Poisson distribution

Here are some examples:

```
using CUDArt
using CURAND

# generate CudaArray of 10 Float64 numbers and copy them to host to view
d_a = curand(10)
to_host(d_a)

# generate array of Float32 numbers
d_a = curand(Float32, 10)
to_host(d_a)
```

These examples use default random number generator of type `CURAND_RNG_PSEUDO_DEFAULT`, you can also create custom generator and use in your calls: 

```
rng = create_generator(CURAND_RNG_QUASI_DEFAULT)

# create array of 10 uniformly distributed numbers using custom RNG
d_a = curand(rng, Float64, 10)

# create array of 10 normally distributed numbers with mean 0. 
# and standard deviation .5 using cutsom RNG
d_a = curandn(rng, Float64, 10, 0, .5)
```

cuRAND support following RNG types (see cuRAND documentation for details):

```
CURAND_RNG_TEST
CURAND_RNG_PSEUDO_DEFAULT	
CURAND_RNG_PSEUDO_XORWOW	
CURAND_RNG_PSEUDO_MRG32K3A
CURAND_RNG_PSEUDO_MTGP32
CURAND_RNG_PSEUDO_MT19937
CURAND_RNG_PSEUDO_PHILOX4_32_10
CURAND_RNG_QUASI_DEFAULT
CURAND_RNG_QUASI_SOBOL32
CURAND_RNG_QUASI_SCRAMBLED_SOBOL32
CURAND_RNG_QUASI_SOBOL64
CURAND_RNG_QUASI_SCRAMBLED_SOBOL64
```

## Low-level API

There's also low-level API that wraps most functions from original `curand.h`. E.g. you can set a seed for a pseudo generator as follows: 

```
rng = create_generator()

set_pseudo_random_generator_seed(rng, 100)

# set seed for default RNG
set_pseudo_random_generator_seed(CURAND._rng, 100)
```

## Troubleshooting

#### RNG initialization failure after `CUDArt.device_reset()`

**Problem:** `curand()` fails with `CURAND_STATUS_INITIALIZATION_FAILED (203)` after `device_reset()` or `devices(...) do ... end` from CUDArt.jl

**Solution:** For convenience, CURAND.jl creates a default instance of random number generator and uses it for all short-form functions. Resetting default device invalidates this RNG, so if you plan to reinitialize device in some point, you have to use explicit generator. E.g. instead of:

```
# somewhere after device_reset() 
curand(Float64, 10)
```

you need to use:

```
# somewhere after device_reset() 
rng = create_generator()
curand(rng, Float64, 10)
```

If error persists, it may be an issue with CUDA version: at the moment it's known that CUDA 7.0 doesn't work, while 6.5 and 7.5 work fine. See [#3](https://github.com/JuliaGPU/CURAND.jl/issues/3) and [this thread](https://groups.google.com/forum/#!topic/julia-users/mJjjTyU7cQ0) for details. 


## TODO

Following functions are not implemented yet: 

 * `curandSetStream`
 * `curandGetDirectionVectors32`
 * `curandGetScrambleConstants32`
 * `curandGetDirectionVectors64`
 * `curandGetScrambleConstants64`

