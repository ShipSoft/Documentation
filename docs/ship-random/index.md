# ship-random

Header-only random number helpers shared by aegir, aegir-genie, Shannon and
Trout. Its only dependency is
[Random123](https://github.com/DEShawResearch/random123).

Repository: [ShipSoft/ship-random](https://github.com/ShipSoft/ship-random)

API reference: [Doxygen](https://shipsoft.github.io/ship-random/)

## `SHiP::random::PhiloxRng`

A counter-based generator built on Philox 4x32. It keeps no shared state:
each event (or spill, or hit) gets a fresh instance from the seed, a stream
key and a counter offset. Results are reproducible and do not depend on the
number of threads.

```cpp
#include <SHiP/random/philox_rng.hpp>

// seed, stream key, sub-stream (e.g. the event number)
SHiP::random::PhiloxRng rng{seed, 0xBEEFCAFE, event_number};
double const u = rng.uniform();            // [0, 1)
double const g = rng.gaussian(0.0, 1.0);   // mean, sigma
```

Give each independent purpose (each generator, each detector) its own stream
key so the sequences stay uncorrelated under the same seed.

The drawn sequences are part of the physics output of every package using
the class. The tests pin them bit for bit, and any change to them needs a new
minor version.

## Using it

Add `ship-random` from [prefix.dev/ship](https://prefix.dev/channels/ship),
then in CMake:

```cmake
find_package(SHiPRandom 0.1 REQUIRED)
target_link_libraries(my_target PRIVATE SHiP::Random)
```

To build and test the repository itself, run `pixi run test`.
