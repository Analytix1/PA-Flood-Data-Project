# Flood Risk Analysis in Pennsylvania

## Project Title  
**Substation Flood Risk Mapping**

---

### Name(s) of Individual or Team Members  
**Ajani “AJ” Adovor**

---

### Short Summary  
In 2021, rain from the remnants of Hurricane Ida caused the Schuylkill River to breach its banks and flood the Vine Street Expressway in Philadelphia, Pennsylvania with 16 feet of water. State and local officials blamed the failure of a major pumping station—whose power supply was cut when its substation was submerged—for the severity of the flooding.

---

### Problem Statement / Research Question / Objective  
**What risk do floods pose to critical electric grid infrastructure, and are these risks accounted for in current flood mapping?**

---

### Datasets (with links, if available)  
- [**Rapid Deployment Network for Flooding in Pennsylvania**](https://www.usgs.gov/data/flood-data-collection-pennsylvania-site-selection-data-development-a-rapid-deployment-network)  
- [**Flood Risk Database**](https://msc.fema.gov/portal/availabilitySearch?addcommunity=420757&communityName=PHILADELPHIA,%20CITY%20OF#searchresultsanchor)  
- [**PA OSM Data via Geofabrik**](https://download.geofabrik.de/north-america/us/pennsylvania.html)

---

### Required Python Packages  
- pandas  
- geopandas  
- numpy  
- pyproj  
- rasterio  
- rioxarray  
- scikit-learn  
- xgboost  
- tensorflow  
- matplotlib  

---

### Planned Methods / Approach  
- Overlay the **Flood Risk Database** with **substation locations**.  
- Compare these results with the **Rapid Deployment Network**.  
- Predict **high exposure areas** for substations at risk of flooding.

---

### Expected Outcomes  
A **grid resilience metric** based on substation exposure to flood risk.

---

### References  
- [Billy Penn: Ida flooding in Philadelphia](https://billypenn.com/2022/09/01/ida-flooding-philadelphia-vine-street-expressway-photos/)  
- [WHYY: Hurricane Ida flooding and infrastructure](https://whyy.org/articles/hurricane-ida-flooding-vine-street-expressway-infrastructure/)  
- [WHYY: Land use and climate change in Philly flood risk](https://whyy.org/articles/its-not-all-about-hurricane-ida-land-use-and-climate-change-drive-philly-flood-risk/)
