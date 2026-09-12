# Monte Carlo Particle Detector Analysis with Python

This project analyzes Monte Carlo simulations of proton hits on a segmented silicon detector for the BL3 neutron-lifetime experiment. Because neutron beta decay produces a proton, the detected proton rate is directly connected to the neutron decay rate used in the lifetime measurement. I examine how changing the detector position affects the spatial distribution of proton hits, how those hits are distributed across the detector’s six-ring, 62-primary-pixel geometry, and whether backscattered protons return to the detector's active area.

The detector-position analysis processes approximately **four million simulated events**, and the backscattering analysis examines an additional **one million events**. The simulation data is stored in ROOT files and analyzed using Python, uproot, NumPy, pandas, and Matplotlib.

![Proton-hit distributions across detector configurations](figures/detector_configuration_comparison.png)

## Related presentation

This analysis grew out of my undergraduate research on BL3 proton tracking with Geant4.

[View the research poster](https://www.pa.uky.edu/~crawford/reu/products_2023/endterm_posters/David_Slone_Poster.pdf)

## Analysis questions

This project investigates four questions:

1. How does moving the detector away from its nominal position affect the location and spread of proton hits?
2. How are proton rates distributed across the detector’s six concentric rings?
3. What pixel-level variation is hidden when detector response is summarized using ring averages?
4. In a simplified backscattering simulation, how often do proton trajectories finish outside the detector's 50 mm active radius?

## Key findings

* Moving from the nominal to the 15 cm configuration shifted the center of the proton-hit distribution and increased its spatial spread.
* Between the nominal and 15 cm configurations, the standard deviation increased by approximately **30% along x** and **32% along y**.
* The first five detector rings received average rates between approximately **4.65 and 6.89 protons/s per pixel**, while the outermost ring received only **0.033 protons/s per pixel**.
* Pixel-level analysis revealed variation that is not visible from ring averages alone, particularly in the detector’s outer regions.
* In the million-event backscattering run, only **3 final proton positions** were outside the detector's 50 mm radius.

## Analysis workflow

The simulation data is stored in ROOT files. I use `uproot` to read the event data directly into Python and NumPy for the numerical analysis.

The workflow includes:

* processing approximately **1,000,000 simulated events per configuration**;
* selecting events with nonzero silicon energy deposition (`SiEdep > 0`);
* transforming global hit coordinates into a detector-centered coordinate system;
* comparing proton-hit distributions at the **Nominal, 3 cm, 6 cm, and 15 cm** configurations;
* converting Cartesian hit coordinates into radial and angular coordinates;
* assigning nominal-position hits to **6 detector rings and 62 individual pixels** using vectorized NumPy operations;
* estimating ring- and pixel-level proton rates using a normalization of **280 protons/s**;
* using the recorded positions and momenta of backscattered protons as initial conditions for new Geant4 runs;
* sampling possible energies and directions for protons leaving the detector surface; and
* comparing their initial and final positions to determine whether they return within the detector's active radius.

## Detector geometry

<table>
  <tr>
    <td width="60%">
      The detector is divided into six concentric rings with radial boundaries
      at <code>3.7, 9.1, 15, 24, 35, and 50 mm</code>.
      <br><br>
      The rings contain <code>1, 5, 10, 20, 10, and 16 pixels</code>, for a
      total of <strong>62 primary detector pixels</strong>. Angular offsets
      determine how pixels are positioned within each ring.
      <br><br>
      The physical detector also includes one small auxiliary pixel near the
      outer edge, giving <strong>63 physical pixels</strong> in total. That
      additional pixel is not represented in the six-ring drawing or
      pixel-rate analysis.
    </td>
    <td width="40%" align="center">
      <img src="figures/detector_ring_geometry.png"
           alt="BL3 segmented silicon-detector geometry"
           width="280">
      <br>
      <sub>Six-ring pixel pattern used in the analysis. The auxiliary
      outer-edge pixel is not shown.</sub>
    </td>
  </tr>
</table>

## Spatial-distribution results

For the spatial comparison, I select events with nonzero silicon energy deposition and display hits within the detector’s **50 mm outer radius**. All four histograms use the same coordinate range, binning, and logarithmic count scale so that the configurations can be compared directly.

| Configuration | SiEdep > 0 events | x mean (mm) | y mean (mm) | x std (mm) | y std (mm) |
| ------------- | ----------------: | ----------: | ----------: | ---------: | ---------: |
| Nominal       |           921,380 |      3.0896 |      0.5265 |    13.1289 |    12.4572 |
| 3 cm          |           920,867 |      2.9834 |      1.5073 |    13.3788 |    12.6537 |
| 6 cm          |           921,476 |      4.0090 |      3.1669 |    13.8189 |    13.0452 |
| 15 cm         |           921,449 |      7.0866 |     15.6059 |    17.0067 |    16.4103 |

The number of events producing silicon energy deposition remains nearly constant across the four configurations. However, the location and shape of the hit distribution change as the detector is displaced.

The largest change occurs in the 15 cm configuration. Relative to the nominal position, the mean x-coordinate increases from **3.09 to 7.09 mm**, while the mean y-coordinate increases from **0.53 to 15.61 mm**. The x and y standard deviations increase by approximately **30% and 32%**, respectively, showing that the distribution becomes substantially broader.

## Ring-level analysis

At the nominal detector position, I convert each valid hit into radial and
angular coordinates and assign hits to the detector's six rings.

![Average proton rate per pixel by detector ring](figures/ring_average_rates.png)

| Ring | Number of pixels | Hit count | Average hits per pixel | Protons/s per pixel |
| ---: | ---------------: | --------: | ---------------------: | ------------------: |
|    1 |                1 |    22,401 |               22,401.0 |              6.8075 |
|    2 |                5 |   113,354 |               22,670.8 |              6.8895 |
|    3 |               10 |   224,332 |               22,433.2 |              6.8173 |
|    4 |               20 |   406,465 |               20,323.3 |              6.1761 |
|    5 |               10 |   153,070 |               15,307.0 |              4.6517 |
|    6 |               16 |     1,758 |                  109.9 |              0.0334 |

The first three rings receive similar average proton rates per pixel. The rate begins to decrease in Rings 4 and 5 and falls sharply in Ring 6, indicating that relatively few nominal-position hits reach the detector’s outermost region.

## Pixel-level analysis

Ring averages can conceal differences among individual pixels. To examine this structure, I assign every valid nominal-position hit to one of the detector’s 62 pixels.

The implementation uses vectorized NumPy operations rather than an event-by-event Python loop. Pixel assignments are calculated simultaneously within each detector ring using radial boundaries, angular offsets, and the number of pixels in that ring.

![Nominal proton rate per detector pixel](figures/nominal_pixel_rates.png)

The pixel-level results reveal variation that is hidden by ring averages, especially in the outer portions of the detector. This demonstrates why analyzing the detector at both ring and pixel resolution provides a more complete description of its simulated response.

## Backscattering analysis

<table>
  <tr>
    <td width="60%">
      Protons that reach the silicon detector are not always immediately
      stopped. A proton can scatter from the detector surface before depositing
      all of its energy, which makes backscattering an important check on
      proton-detection efficiency.
      <br><br>
      To study this effect, we modified the Geant4 simulation to record the
      position and momentum of protons as they backscattered from the silicon
      detector. We then used those states as initial conditions for a new
      simulation run that sampled possible energies and directions for protons
      leaving the detector surface.
    </td>
    <td width="40%" align="center">
      <img src="figures/proton_backscattering_paths.png"
           alt="Simulated proton trajectories backscattering from the silicon detector"
           width="280">
      <br>
      <sub>Geant4 visualization of 100 protons backscattering from the detector.
      The red surface represents the silicon detector, and the colored lines
      show the simulated proton paths.</sub>
    </td>
  </tr>
</table>

The Python analysis compares the initial backscatter positions with the final
proton positions, selects events with nonzero silicon energy deposition, and
determines whether each final position lies within the detector's 50 mm outer
radius.

![Initial and final proton backscattering distributions at the nominal detector position](figures/backscattering_initial_final_comparison.png)

In the million-event simulation run, only **3 final proton positions** were
found outside the detector radius. This indicates very small geometric losses
under the assumptions of the simplified backscattering model.

This result describes proton transport after a backscattered proton is
initialized at the detector surface. It is not the physical probability
that an incident proton will backscatter from silicon.

## Implementation

The notebooks use Python and NumPy for:

* loading and validating ROOT data;
* selecting detector-hit events;
* transforming coordinates;
* calculating summary statistics;
* assigning hits to detector rings;
* mapping hits to individual pixels;
* calculating normalized proton rates;
* comparing initial and final backscatter positions;
* measuring geometric losses from backscattered trajectories;
* generating portfolio figures.

## Repository structure

```text
particle-detector-analysis/
├── detector_analysis.ipynb
├── backscatter_analysis.ipynb
├── README.md
├── requirements.txt
├── .gitignore
└── figures/
    ├── backscattering_initial_final_comparison.png
    ├── detector_configuration_comparison.png
    ├── detector_ring_geometry.png
    ├── proton_backscattering_paths.png
    ├── ring_average_rates.png
    └── nominal_pixel_rates.png
```


## Data availability

The detector-position analysis uses four ROOT simulation files corresponding
to the Nominal, 3 cm, 6 cm, and 15 cm detector configurations. The
backscattering analysis uses an additional million-event ROOT simulation created
from the modified Geant4 model.

These simulation files are not distributed with this repository. The
notebooks therefore serve as a documented record of the analysis and
includes the saved tables, numerical results, and figures generated from
the simulations. The backscattering notebook similarly documents its event
selection, detector-acceptance calculation, and reported results.

## Tools

**Python · NumPy · pandas · Matplotlib · uproot · Jupyter · ROOT**
