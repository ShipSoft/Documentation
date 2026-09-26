# Samples

> [!WARNING]
> **Dimuon boost and sample composition**
>
> Samples generated with `dimuonboost > 1` are **not** representative of the
> relative fluxes of the different particle species. In particular the
> neutrino flux is heavily diluted: at `dimuonboost = 100` neutrinos make up
> only a few percent of the particles in the file (~28k out of ~660k),
> against roughly half at `dimuonboost = 1` (~23k out of ~45k).
>
> Perhaps less obviously, the *unboosted* samples also contain a substantial
> dimuon component. Extrapolating linearly from the two boost settings gives
> **~14% of all particles and ~28% of the non-neutrino particles** in the
> `dimuonboost = 1` case. Anyone using these samples for absolute rates should
> apply the appropriate event weights rather than assuming the unboosted sample
> is unbiased.
