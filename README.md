# Free tickets (let's dream)
## A study of potential gratuity in the parisian subway
### 📝 Summary & objective
I made this analysis along with 3 other teammates at Le Wagon. This is the final project we presented on our last day in front of all the bootcamp participants. We played the role of external consultants that make recommendations to Ile-de-France Mobilités (IDFM), aka the entity that manages public transportation in the region.  

Our objective was to assess the consequences of partial / complete gratuity in the parisian subway. 
We chose 3 axis to develop our analysis: peak periods, budget and environment:
- 👫 <ins>**Peak periods**:</ins> how can gratuity impact rush hours and frequentation ? Can IDFM absorb the extra passengers ? **Main KPI : number of validations**
- 💰 <ins>**Budget**:</ins> How much will the shortfall cost if IDFM does not sell any more tickets ? What is the best strategy between partial or total gratuity ? **Main KPI: estimation of the loss of income**
- 🌱 <ins>**Environment**:</ins> Can we estimate the impact of gratuity on CO2 emissions in the Paris area ? **Main KPI: tons of CO2 emissions**

### ⚠️ Disclaimer
This analysis is made with public data only: we did know that the analysis is not completely suited for the "real" world. We did not have any information about IDFM's capacity of evolution in terms or infrastructure, nor about their detailed budget. However, I think our presentation is still interesting both for the data manipulation we made and the content that came out of it. 

### 📥 Data source and perimeter of the analysis
**Main data source** : [Plateforme Régionale d'Information pour la Mobilité](https://prim.iledefrance-mobilites.fr/fr) (PRIM) for frequentation data and types of tickets  
**Secondary data source** : [Portail OpenData d'AirParif](https://data-airparif-asso.opendata.arcgis.com/) for data about greenhouse gas emissions.

The data comes out as csv files, one for 2022 data and one for 2023. Railroad network (subway, RER) and road network (bus, tram) are also separated into different files. 
The analysis goes from June 2022 to June 2023 to get one complete year of data. 
Due to the short time we had to analyze the data, we decide to narrow our perimeter to **the inner Paris railroad network**, excluding de facto subway stations outside of Paris and bus / tram data.   
