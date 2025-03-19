# Free tickets (let's dream)
## A study of potential gratuity in the parisian subway

I made this analysis along with 3 other teammates at Le Wagon. This is the final project we presented on our last day in front of all the bootcamp participants. We played the role of external consultants that make recommendations to Ile-de-France Mobilités (IDFM), i.e. the entity that manages public transportation in the region.  

➡️ Read the data transformation presentation [here](https://docs.google.com/presentation/d/1oL-Tq8IW-vz1SThDDeRLEMGLRJV-Heb2UD_2cxzZhlA/edit?usp=sharing) (in French)  
➡️ Access the final dashboard we presented [here](https://lookerstudio.google.com/u/1/reporting/2155cd6b-8a4a-4ff5-a74e-8d4efbb9c34c/page/p_3xqoufdccd) (in French)

### ⚠️ Disclaimer
This analysis is made with public data only: we did know that the analysis is not completely suited for the "real" world. We did not have any information about IDFM's capacity of evolution in terms or infrastructure, nor about their detailed budget. However, I think our presentation is still interesting both for the data manipulation we made and the content that came out of it. 

### Table of contents
- [Summary & objective](#-summary--objective)
- [Data source and perimeter of the analysis](#-data-source-and-perimeter-of-the-analysis)

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

<ins>Here are the cleaning actions we performed on the **validations** table, i.e. our main table</ins>
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

<ins>After cleaning the data, the **validations** and the **profil_horaire** table schemas are as follows:</in>  
![valid_clean](img/validations_clean.png)  ![profil_clean](img/profil_clean.png)  

**JOUR** = date (day)   
**LIBELLE_ARRET** = name of the subway station  
**CATEGORIE_TITRE** = type of subway card (full price, monthly subscription, student price...)   
**NB_VALID** = number of validations  
**CAT_JOUR** = category of day as defined by IDFM (weekday, week-end, bank holiday, school vacation)  
**TRNC_HORR_60** = hour of the day (60-minute periods)   
**pourc_validations** = percentage of validations counted in the specific hour of the day  

### 🪄 Data Transformation with Python (Pandas library): create a first table containing the hourly validations

- **Group by JOUR, LIBELLE_ARRET, CATEGORIE_TITRE, sum of NB_VALID** - *validations table*
  ```Python
  validations = validations.groupby(['JOUR','LIBELLE_ARRET','CATEGORIE_TITRE'], as_index=False).sum()
  ```
- **Add type of day using a join with the type_jour table** - *validations table*
  ```Python
  validations = validations.merge(type_jour, on='JOUR', how='left')
  ```
- **Create a join key between validations and profil horaire** - *validations AND profil horaire table*
  ```Python
  validations['station_jour'] = validations['LIBELLE_ARRET'] + validations['CAT_JOUR']
  profil['station_jour'] = profil['LIBELLE_ARRET'] + profil['CAT_JOUR']
  ```
- **Add TRNC_HORR_60 and pourc_validations to the validations table with a LEFT JOIN on profil horaire**
  ```Python
  validations = validations.merge(profil, on='station_jour', how='left')
  ```
- **Calculate the number of validations by hour of the day** - *validations table*
  ```Python
  validations['valid_horaire'] = validations['NB_VALID']*validations['pourc_validations']/100
  #format this new column as an integer because validations can only be an integer:
  validations['valid_horaire'] = validations['valid_horaire'].astype(int)
  ```
- **Rename the table**
  ```Python
  valid_horaire = validations
  ```
  We now have a new enriched table **valid_horaire** as follows, that we can export in BigQuery:  
  ![valid_horaire](img/valid_horaire.png)  

- **Export in BigQuery**
  ```Python
  from pandas_gbq import to_gbq
  to_gbq(df, f'free_tickets.valid_horaire', project_id=training-431014, if_exists='append')
  ```

### 🪄 Data transformation with SQL (BigQuery): create a second table with the calculation of the gap of revenue  

Part of the transformation we did has been made with Python, the other one has been made in BigQuery with SQL, because we were working together at the same time, and each of us was experiencing different techniques. 

The objective here is to get information about the prices of the different tickets, in order to estimate the shortfall due to the potential gratuity (i.e. the money IDFM is going to lose if nobody pays). 
The first thing we had to do was to determine the average price of a validation for each type of ticket. The idea was to take the data we had on one year, and calculate the daily average price of a validation by type. 
That is how we came out with this table **prix_moyen_validation**:  
![avg_price](img/avg_price.png)  

We also worked with the cleaned **validations** table we exported in BQ from our Python notebook:  
![valid_clean](img/validations_clean.png)  

- **Add the unit price of a validation to the validations table**
  ```SQL
  SELECT
  validations.*,
  prix_moyen_validation.prix_par_validation
  FROM validations LEFT JOIN prix_moyen_validation USING CATEGORIE_TITRE
  ```

- **Add the total shortfall ('GAP_REVENU') to the validations table**
  ```SQL
  SELECT
  *,
  NB_VALID * prix_par_validation AS GAP_REVENU
  FROM validations
  ``` 

- **Add geolocation data to the validations table**  
With a left join on the name of the subway stations ('LIBELLE_ARRET'), we also added the data coordinates of each station (coming from a       secondary table **emplacement_gares_ligne_metro**, see ERD), because it could possibely be useful for graphs in Looker Studio. 
  ```SQL
  SELECT
  validations.*,
  geo.Geo_Shape,
  geo.Geo_Point
  FROM validations LEFT JOIN emplacement_gares_ligne_metro AS geo USING LIBELLE_ARRET
  ```

So we ended with another enriched **validations** table containing the financial information we need: 
![enriched_valid](img/enriched_valid.png)  


At the end of our data transformation, we had 2 enriched tables, **valid_horaire** and **validations**, that we exported in Looker Studio to structure our analysis. 

### 📊 Data Vizualisation and Analysis with Looker Studio

<ins>**FIRST AXIS : IMPACT OF THE GRATUITY ON FREQUENTATION**</ins> 👫 

The first thing to say about frequentation is that we have to take into account several parameters that have a big impact on the analysis: the number of validations depends on the location (center vs border of Paris), on the day of the week (more traffic during weekdays), and on the hour of the day (the peak hours being 8am and 6pm especially on weekdays), as we can see in these graphs below:

<img src="https://github.com/charlottedhu/free_tickets_paris/blob/b0d25a252ba2aac6807b01237abb319fae177ba1/img/valid_station.png" width=50% height=50%>  

![affluence](img/affluence.png)

The idea is to anticipate what will be the total frequentation by year and average by day if it increases with gratuity. We made 3 hypothesis: low, medium and high increase. This is what we have in the capture below.   
- **LOW**: Freqentation increases by 2% ➡️ It represents 19.5 millions more people in the subway in a year, i.e. 54K more people a day on average  
- **MEDIUM**: Frequentation increases by 5% ➡️ It represents 48.7 millions more people in the subway in a year, i.e. 136K more people a day on average  
- **HIGH**: Frequentation increases by 10% ➡️ It represents 97.4 millions more people in the subway in a year, i.e. 272K more people a day on average  

![affluence_prev](img/affluence_prev.png)  

What is interesting is that we can filter these figures and graphs by type of day and / or by subway station. It can be useful for IDFM to anticipate which are the days, hours or stations that will be jammed and could require additionnal service (which means capacity and budget). It is a way to know if they can offer gratuity given their capacity and budget.  
For example, these are the figures we get when we filter on week-ends + bank holidays only (school vacation week-ends included):  

![affluence_we](img/affluence_we.png)  

We do not have access to capacity and infrastructure budget, so the recommendations we made could be more precise, but based on the insights we have, and because week-ends represent around 20% of the traffic, we can say that if IDFM wants to test gratuity, it could be interesting to focus this test on week-ends only. Some cities already offer this type of gratuity, like Nantes for example, although the city is way smaller than Paris.  

<ins>**SECOND AXIS : IMPACT OF THE GRATUITY ON BUDGET**</ins> 💰 

