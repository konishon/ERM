# Extreme rainfall monitoring, will it trigger a rainfall?

Extreme Rainfall Monitoring (ERM) is an experimental tool developed by the United Nations [World Food Programme](https://www.wfp.org/countries/indonesia) in Indonesia written in [Google Earth Engine](https://earthengine.google.com) (GEE) platform. ERM are able to inform where is the estimated location of extreme rainfall and its impact to population and crop in the last 5-days and forecast up to 5-days ahead based on selected date.

Some of the input was prepared via different platform: ArcGIS Pro, R Statistics, and Excel. ERM module is part of Vulnerability Analysis and Monitoring Platform for Impact of Regional Events ([VAMPIRE](http://vampire.pulselabjakarta.org) - now called PRISM) hazard monitoring module. Some of the analysis has been modified and adjusted to GEE based on data availability and considering the computation resources within GEE. While the methodology written in this document still reflecting what we developed in VAMPIRE.

We are actively migrating the current GEE workflows to FOSS technologies.

This is the single-source documentation for ERM. It is intended to be a thorough, living document detailing the background, objective, data, method and output of ERM. If you find a mistake, please open an issue.

ERM model developed by Benny Istanto and [Prof. Rizaldi Boer](https://scholar.google.com/citations?hl=en&user=jTPXEp8AAAAJ) of Climatology Laboratory - [Bogor Agricultural University](https://ipb.ac.id) as Scientific Advisor.

![ERM1](./docs/img/erm1.png)

![ERM2](./docs/img/erm2.png)


### Documentation

[https://wfpidn.github.io/ERM](https://wfpidn.github.io/ERM)


### Demo

[https://bennyistanto.users.earthengine.app/view/wfpid-erm](https://bennyistanto.users.earthengine.app/view/wfpid-erm)


### Running the Python notebooks

Python notebooks for data access and geospatial exploration live in `script/`. To run them locally:

1. Install [Poetry](https://python-poetry.org/) and ensure you have a working Python 3.11+ environment.
2. From the repository root, install dependencies: `poetry install` (add `--with dev` if you need the optional STAC/Planetary Computer tooling).
3. Start an environment and Jupyter: `poetry run jupyter lab` (or `poetry run jupyter notebook`).
4. Open the notebook you need from `script/` (for example `erm_geemap.ipynb` or `erm_stac.ipynb`).
5. For notebooks that call Google Earth Engine, make sure your account is enabled and run `earthengine authenticate` once before executing the cells.


### Contact

For further information about Extreme Rainfall Monitoring, please contact:

**Benny Istanto**<br>
Earth Observation and Climate Analyst<br>

Vulnerability Analysis and Mapping Unit<br>
UN World Food Programme<br>
Jakarta, Indonesia<br>

E. [benny.istanto@wfp.org](mailto:benny.istanto@wfp.org)<br>

![VAM](./docs/img/WFP_newVAM_Logo.png)
