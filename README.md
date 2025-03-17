# Free tickets (let's dream)
## A study of potential gratuity in the parisian subway

I made this analysis along with 3 other teammates at Le Wagon. This is the final project we presented on our last day in front of all the bootcamp participants. We played the role of external consultants that make recommendations to Ile-de-France Mobilités (IDFM), aka the entity that manages public transportation in the region.  

➡️ Read the data transformation presentation [here](https://docs.google.com/presentation/d/1oL-Tq8IW-vz1SThDDeRLEMGLRJV-Heb2UD_2cxzZhlA/edit?usp=sharing) (in French)  
➡️ Access the final dashboard we presented [here](https://lookerstudio.google.com/u/1/reporting/2155cd6b-8a4a-4ff5-a74e-8d4efbb9c34c/page/p_3xqoufdccd) (in French)

### ⚠️ Disclaimer
This analysis is made with public data only: we did know that the analysis is not completely suited for the "real" world. We did not have any information about IDFM's capacity of evolution in terms or infrastructure, nor about their detailed budget. However, I think our presentation is still interesting both for the data manipulation we made and the content that came out of it. 

### 📝 Summary & objective
Our objective was to assess the consequences of partial / complete gratuity in the parisian subway. 
We chose 3 axis to develop our analysis: peak periods, budget and environment:
- 👫 <ins>**Peak periods**:</ins> how can gratuity impact rush hours and frequentation ? Can IDFM absorb the extra passengers ? **Main KPI : number of validations**
- 💰 <ins>**Budget**:</ins> How much will the shortfall cost if IDFM does not sell any more tickets ? What is the best strategy between partial or total gratuity ? **Main KPI: estimation of the loss of income**
- 🌱 <ins>**Environment**:</ins> Can we estimate the impact of gratuity on CO2 emissions in the Paris area ? **Main KPI: tons of CO2 emissions**

### 📥 Data source and perimeter of the analysis
**Main data source** : [Plateforme Régionale d'Information pour la Mobilité](https://prim.iledefrance-mobilites.fr/fr) (PRIM) for frequentation data and types of tickets  
**Secondary data source** : [Portail OpenData d'AirParif](https://data-airparif-asso.opendata.arcgis.com/) for data about greenhouse gas emissions.

- The data comes out as csv files, one for 2022 data and one for 2023. Railroad network (subway, RER) and road network (bus, tram) are also separated into different files.  
- The analysis goes from June 2022 to June 2023 to get one complete year of data.  
- Due to the short time we had to analyze the data, we decide to narrow our perimeter to **the inner Paris railroad network**, excluding de facto subway stations outside of Paris and bus / tram data.
- 2 main types of tables: **validations** (number of validations by ticket categories, days ands stations) and **profil horaire** (percentage of validations by hour, type of day, and stations) - See ERD below
- secondary tables: geolocation, information about tickets, type of days (week-days, week-ends, bank holidays, school vacation) - See ERD below

![ERD](img/ERD_free_tickets.png)

### 🚿 Data cleaning of the main data source with Python (Pandas library)

<ins>Here are the cleaning actions we performed on the **validations** table, aka our main table</ins>
- **Concatenate the 2022 and the 2023 tables**
  ```Python
  df_concat = pd.concat([df_final,df2_final], axis=0).reset_index(drop=True)
  ```
- **Treat null values**  
  We chose to drop null values because it was only a very small part of the data, among several millions of lines in the table, so the impact would be minimal.
  ```Python
  df_clean = df_concat.dropna()
  ```
- **Filter on inner Paris data**  
  In order to apply this filter, we had to keep only lines where the variable CODE_STIF_TRNS equals 100
  ```Python
  mask_stif_trns = df_clean['CODE_STIF_TRNS']==100
  df_final = df_clean[mask_stif_trns]
  ``` 
- **Drop useless columns**  
  Removing columns containing internal codes or references that are not relevant for filtering or analysis
  ```Python
  df_final = df_final.drop(columns=df_final[['CODE_STIF_ARRET','lda','CODE_STIF_RES','CODE_STIF_TRNS']])
  ``` 
- **Format data types**  
  Changing the 'JOUR' column in date format, and the 'NB_VALID' in numeric format
  ```Python
  df_final['JOUR'] = pd.to_datetime(df_final['JOUR'],format=%d/%m/%Y)
  df_final['NB_VALID'] = df_final['NB_VALID'].astype(int)
  ```

<ins>As for the other tables:</ins> 
- We performed similar actions on the **profil horaire** table (concatenate, drop null values, filter and reformat)
- The other tables were way smaller, and will be useful to qualify the data (type of day, type of client). These tables were quite clean already. 

### 🪄 Data Transformation
