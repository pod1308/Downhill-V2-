# Methodology and limits

Every source below is read live when the page opens. Nothing is cached,
so no copy in this file should differ from the agency's.

---

## 1. Scores: CalEnviroScreen 5.0

From OEHHA, layer 2 of their feature service:

```
services1.arcgis.com/PCHfdHz4GlDNAhBb/arcgis/rest/services/
  calenviroscreen50results_F_070126_gdb/FeatureServer/2
```

| What | Field |
|---|---|
| Tract ID | `tract` |
| County | `county` |
| Population | `Population`, 2024 five-year ACS |
| Overall percentile | `CIscoreP` |
| Component percentiles | `Pollution_Pctl`, `PopCharPctl` |

23 indicators: eight exposure, six environmental effects, four sensitive
population, five socioeconomic. Each is explained in one line in the app.

**Zeros and blanks.** A 0 means the tract had none of something and was taken
out of that ranking, not that it ranked lowest. A blank is NA: not measured, or
judged unreliable. The app prints "no impact" and "not reported". Tracts with no
overall score are kept and drawn unshaded, because some are designated on
pollution burden alone. Neither 0 nor NA means safe.

**Permitted use**, quoted from OEHHA because it limits everything downstream:
CalEnviroScreen "is designed for screening at statewide, regional, and
tract-level scales, not below the tract level... It is not a substitute for CEQA
cumulative impact determinations, health risk assessments, or analyses requiring
forecasts, thresholds, or before-and-after comparisons."
[CalEnviroScreen 5.0](https://oehha.ca.gov/calenviroscreen/report/calenviroscreen-50)

---

## 2. Designations

### SB 535 disadvantaged communities, 2026

CalEPA's designation, served from the state's ArcGIS organisation:

```
services1.arcgis.com/PCHfdHz4GlDNAhBb/arcgis/rest/services/
  SB_535_Disadvantaged_Communities_2026/FeatureServer/0
```

Fields used: `tract`, `dac_type`. The list runs to 2,704 tracts, so it is read in
two pages of 2,000 and joined to the scores by tract ID.

`dac_type` is shown verbatim, with CalEPA's own wording of the criterion beside
it ([California Climate Investments](https://www.caclimateinvestments.ca.gov/priority-populations)):

| `dac_type` | Criterion |
|---|---|
| CES 5.0 Top 25% | Top 25% of CalEnviroScreen 5.0 overall scores |
| CES 5.0 High Pollution/Low Population | No overall score because of data gaps, but top 5% for pollution burden |
| CES 4.0 Carry-Over DAC | Overlaps a 2022 designated tract by at least 25% |

The designation applies to projects selected on or after 18 September 2026
([CalEPA](https://calepa.ca.gov/programs/dac2026/)). The map is not the record of
designation. For a funding application, confirm on CARB's
[Priority Populations map](https://gis.carb.arb.ca.gov/Portal/apps/experiencebuilder/experience/?id=9736a6e26af84b36b317317917586791).

### Tribal lands

Lands under the control of federally recognized tribes are designated in their
own right, not through a tract score. Layers 1 to 3 of the same service carry
them from three boundary sources:

| Layer | Source | Name field |
|---|---|---|
| 1 | Bureau of Indian Affairs | `TRIBE` |
| 2 | Tribal consultation list | `Name` |
| 3 | Census Bureau | `NAMELSAD` |

The sources overlap, so one place can arrive three times. The app lists it once
with every source that carries it. If one layer fails, the others still draw and
the failure is reported.

The inspector separates tribal land **at the clicked point** from tribal land
**elsewhere in the same tract**. A tract can be many times larger than a
rancheria inside it, and CalEPA includes tribal lands partly because
CalEnviroScreen's data can be limited there, so a tract's scores may not
describe that land.

### AB 617 communities

From CARB's own map service, 19 communities with the year each was selected and
whether it has an emissions reduction plan, an air monitoring plan, or both:

```
gis.carb.arb.ca.gov/hosting/rest/services/
  AB617_Communities_and_Districts_MIL1/MapServer/0
```

[CARB, Community Air Protection](https://ww2.arb.ca.gov/capp)

### AB 1550 low-income communities

Not computed. The test is income at or below 80% of the statewide median or
below the state's low-income threshold, which uses income data CalEnviroScreen
doesn't carry. The app links to CARB's map instead.

---

## 3. Districts

Census Bureau TIGERweb, `Legislative/MapServer`:

| Layer | Districts |
|---|---|
| 2 | 2026 State Assembly |
| 1 | 2026 State Senate |
| 4 | 119th Congress, served now |
| 0 | 120th Congress, from January 2027 |

The inspector queries the clicked point. The area tool fetches one district's
boundary by GEOID (`06` plus the district number, three digits for the
legislature, two for Congress). Census boundaries are for statistical use; for
anything legal, check the Secretary of State or the Citizens Redistricting
Commission.

Some state and federal servers send no CORS header. Every ArcGIS request here
tries `fetch` first and falls back to JSONP, which ArcGIS REST supports with
`&callback=`.

---

## 4. Area summaries

For a county, a district, an AB 617 community, or tracts picked by hand.

- **Assignment.** A tract belongs to an area if its centre point falls inside.
  A tract straddling a boundary counts once.
- **Summed.** Population, tract counts, designated tracts and the residents in
  them.
- **Estimated.** Residents in each age and race group, as each tract's published
  share times its population. ACS margins of error at tract level are wide, one
  study found them about 75% larger on average than the 2000 long form
  ([Spielman, Folch and Nagle 2014](https://pmc.ncbi.nlm.nih.gov/articles/PMC4232960/)),
  and small groups fare worst. This dataset publishes the shares without margins
  of error, so the estimates are labelled approximate.
- **Not averaged.** Percentiles. A mean of ranks is not a rank, and a few extreme
  tracts disappear into a middling mean. The summary shows tract counts in four
  bands, the median, and the range.
- **Concentration.** For each indicator, how many tracts sit at or above the
  state's 90th percentile. Indicators built from facility data are tagged.

The brief and the CSV carry the retrieval date, sources and limits.

---

## 5. Facilities

From EPA's Facility Registry Service, `FRS_INTERESTS`. `REGISTRY_ID` is
non-nullable and the record's primary key, so every site drawn has one and every
compliance link resolves.

| Tier | What it is | On by default |
|---|---|---|
| Major air polluters | Clean Air Act majors | yes |
| Major water dischargers | NPDES majors | yes |
| Superfund and cleanup | Superfund Enterprise Management System | yes |
| Toxic release reporters | TRI filers | yes |
| Power generation | eGRID, emissions markets, greenhouse gas reporting | yes |
| Hazardous waste handlers | RCRA sites | no |
| Every other regulated site | the rest of the register | no |

A site is in a tier because EPA files it under that programme. Tiers rank
regulatory programme, not harm. Coordinates and their accuracy estimates are
EPA's. A point means a site holds or held a permit, or is a documented cleanup
site. It does not mean it is polluting or out of compliance.

### Why the sites are not independent evidence

Six indicators are computed from facility data: Cleanup Sites, Groundwater
Threats, Hazardous Waste, Toxic Releases, Solid Waste, Small Air Toxic Sites. A
facility inside a high-scoring tract is partly why the tract scores that way.
The map names the site and links its record. It never presents proximity as
corroboration.

---

## 6. Interventions: what the map does and doesn't do

### No "remove this facility" simulation

Recalculating a tract's score with one facility subtracted would look precise
and mean little:

1. **Percentiles are relative.** Removing a site changes the statewide
   distribution of that indicator. A new percentile for one tract can't be
   computed without recomputing every tract.
2. **Site indicators are counts, not emissions.** OEHHA weights nearby sites by
   type and distance. Subtracting one lowers a tally. It says nothing about
   anyone's exposure.
3. **Most of the score wouldn't move.** Asthma, cardiovascular disease, low
   birth weight, poverty and housing burden are measured outcomes. A
   recalculation leaves them where they are, though the studies below found
   real health changes after closures.
4. **OEHHA rules it out.** The tool is not a substitute for analyses requiring
   forecasts or before-and-after comparisons.

A defensible site-specific estimate comes from an emissions inventory, a
dispersion model and a health risk assessment. In California that is the
[Air Toxics Hot Spots programme](https://oehha.ca.gov/air/air-toxics-hot-spots):
facilities report emissions, air districts decide which need a health risk
assessment, and assessments follow OEHHA's guidance.

### The studies panel instead

Natural experiments where a source closed, was cleaned up or was retrofitted,
and researchers measured what followed. Each entry states only what the
abstract or the authors' institution reports, and its limits.

| Intervention | Measured result | Source |
|---|---|---|
| 8 California coal and oil plant retirements, 2001–2011 | Preterm birth within 5 km fell from 7.0% to 5.1%; larger fall for non-Hispanic Black and Asian mothers | [Casey et al. 2018, *AJE*](https://pmc.ncbi.nlm.nih.gov/articles/PMC6070091/) |
| Louisville coal retirement plus three SO2 scrubbers, 2013–2016 | Fewer asthma hospitalizations and ER visits; less rescue inhaler use | [Casey et al. 2020, *Nature Energy*](https://www.nature.com/articles/s41560-020-0600-2) |
| 3 coal plants near Chicago closed | Asthma ER visits among children 0–4 down 12% nearby | [Komisarow and Pakhtigian 2021, *AJPH*](https://doi.org/10.2105/AJPH.2021.306155) |
| Shenango coke plant closure, Pittsburgh, 2016 | Nearby SO2 −90%; cardiovascular ER visits −42% immediately (95% CI 33–51%) | [Yu and Thurston 2023, *Environ. Res.: Health*](https://doi.org/10.1088/2752-5309/ace4ea) |
| Utah Valley steel mill closed, 1986–87 | Children's respiratory admissions 2–3x higher in winters with the mill open | [Pope 1989, *AJPH*](https://doi.org/10.2105/AJPH.79.5.623) |
| ~1,600 TRI plant openings and closings | Air effects within ~1 mile; ~3% higher low birth weight risk within 1 mile while operating; openings cut home values ~11% within 0.5 mile | [Currie et al. 2015, *AER*](https://www.aeaweb.org/articles?id=10.1257/aer.20121656) |
| Superfund cleanups, five states, 1989–2003 | Before cleanup, 20–25% higher risk of congenital anomalies within 2 km | [Currie, Greenstone and Moretti 2011, *AER*](https://www.aeaweb.org/articles?id=10.1257/aer.101.3.435) |
| Superfund cleanups at 321 sites, 1990–2000 | Home values within 3 miles up 18.6–24.5%; none detected in metro LA | [Gamper-Rabindran and Timmins 2013, *JEEM*](https://ideas.repec.org/a/eee/jeeman/v65y2013i3p345-360.html) |
| First 400 Superfund cleanups vs 290 near misses | No detectable effect on local housing price growth | [Greenstone and Gallagher 2008, *QJE*](https://www.nber.org/papers/w11790) |
| Port of Oakland truck retrofit and replacement rule | Fleet black carbon emissions −54%, NOx −41% | [Dallmann, Harley and Kirchstetter 2011, *ES&T*](https://doi.org/10.1021/es202609q) |
| San Pedro Bay Ports Clean Air Action Plan, Port of LA | Diesel particulate −90%, NOx −73%, SOx −98% since 2005 (self-reported inventory) | [Port of Los Angeles, Oct 2025](https://portoflosangeles.org/references/2025-News-Releases/news_101625_air_emissions) |
| 2,656 Georgia school bus retrofits | Better aerobic capacity; English score gains, smaller math gains | [Austin, Heutel and Kreisman 2019, *Econ. Educ. Rev.*](https://www.nber.org/papers/w25641) |
| Atlanta Olympics traffic measures, 1996 | Peak ozone −27.9%; child asthma acute care −41.6% in Medicaid | [Friedman et al. 2001, *JAMA*](https://doi.org/10.1001/jama.285.7.897) |
| Stockholm congestion tax | Pollution −5 to 15%; fewer acute asthma attacks in young children | [Simeonova et al. 2021, *J. Human Resources*](https://www.nber.org/papers/w24410) |
| E-ZPass at toll plazas, NJ and PA | Prematurity −10.8%, low birth weight −11.8% within 2 km | [Currie and Walker 2011, *AEJ: Applied*](https://www.aeaweb.org/articles?id=10.1257/app.3.1.65) |
| Dublin coal sales ban, 1990 | Black smoke −70%; respiratory deaths −15.5%, cardiovascular −10.3% | [Clancy et al. 2002, *Lancet*](https://doi.org/10.1016/S0140-6736(02)11281-5); [2013 follow-up](https://www.healtheffects.org/publication/effect-air-pollution-control-mortality-and-hospital-admissions-ireland) |
| Hong Kong fuel sulphur cap, 1990 | Annual death trend −2.1% all causes, −3.9% respiratory | [Hedley et al. 2002, *Lancet*](https://doi.org/10.1016/S0140-6736(02)11612-6) |

Clicking a facility offers the studies for its kind of source. None of them is a
forecast for that facility. They differ in place, era, source type and outcome,
and the magnitudes don't transfer.

---

## 7. No property price layer

A price beside a facility invites the reading that the facility set the price.
Prices move with schools, transit, zoning and the wider market, and nothing on
screen separates those. What the evidence says is in the studies panel: plant
openings were followed by lower nearby values, while Superfund cleanups showed
no detectable effect on price growth. CalEnviroScreen's housing burden
indicator, the share of households both low income and severely burdened by
housing costs, is already in every tract's scores.

---

## 8. Water and the downstream trace

Channels from OpenStreetMap's `waterway` layer in the basemap's vector tiles.
OSM draws a waterway from source to mouth, so point order is flow order. At each
junction the trace takes a reach starting within 130 m of where the last ended,
favouring larger channels.

| Channel | Assumed velocity |
|---|---|
| River | 0.8 m/s |
| Canal | 0.6 m/s |
| Stream | 0.45 m/s |
| Storm drain, open | 0.9 m/s |
| Ditch | 0.35 m/s |
| Tidal channel | 0.25 m/s |
| Storm sewer, piped | 1.2 m/s |

Mid-range figures for the channel type, not measurements. Read a result as an
order of magnitude. No terrain: the first leg is a distance to the nearest
channel, not a route, and in a city it is mostly storm sewer, which can cross a
surface watershed boundary. Good for showing a site drains to a named water
body. Not good enough to say a discharge reached it; that takes monitoring.

---

## 9. Known gaps

- No air monitoring data. Air quality reaches the map only through
  CalEnviroScreen.
- No AB 1550 low-income layer. CARB's map has it.
- No drinking water system or groundwater basin boundaries.
- No enforcement or violation history in the app itself. Each facility links to
  its EPA compliance report, which has it.
- No change over time, by design.
- Tract boundaries are generalised and unsuitable for parcel-level work.
- Area summaries assign tracts by centre point, which misplaces slivers of
  tracts that straddle a boundary.

---

## 10. Checking any of this

- **Scores.** Inspect a tract and use its check link to OEHHA's official map.
  Full data and dictionary: [data.ca.gov](https://data.ca.gov/dataset/calenviroscreen-5-0).
- **Designation.** CARB's [Priority Populations map](https://gis.carb.arb.ca.gov/Portal/apps/experiencebuilder/experience/?id=9736a6e26af84b36b317317917586791)
  or the [SB 535 dataset](https://data.ca.gov/dataset/sb-535-disadvantaged-communities-2026).
- **Facilities.** The linked EPA detailed facility report.
- **Studies.** The linked paper.
- **Any service.** Open the URLs above in a browser. Each is a public ArcGIS
  endpoint that returns its own schema.

If any of those disagree with this map, they are right. Please open an issue.
