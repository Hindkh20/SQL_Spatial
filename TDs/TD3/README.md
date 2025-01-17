# Spatial Database  
## TD-3: Spatial SQL Functions  

### Objective:  
Use PostGIS to manipulate spatial data and answer spatial analysis questions based on SQL queries.  

---

## Scenario:  
In this exercise, you will determine how many parcels in Florida are within a 1 km radius around a fire point. The fire point is defined as the centroid of a given parcel. You will use the Florida parcel shapefile available at this link.  

### Step 1: Import the Shapefile into PostGIS  
1. Download the shapefile.  
2. Open **PostGIS Shapefile Import/Export Manager**.  
   - On Windows, press the Windows key and search for `shap`. You should see the **PostGIS Shapefile Import/Export Manager** tool.  
3. Select your Florida parcel shapefile.  
4. Import it into a PostgreSQL table.  

---

### Step 2: Verify the Import in pgAdmin  

- **SQL Command:**  
  ```sql
  SELECT COUNT(*) FROM parcels;
  ```  
  **Response:** This query returns the total number of parcels imported into the `parcels` table.  

- **SQL Command:**  
  ```sql
  SELECT * FROM parcels LIMIT 5;
  ```  
  **Response:** This query returns the first 5 rows from the `parcels` table.  

---

### Step 3: Select the Fire Point  

- **SQL Command:**  
  ```sql
  SELECT ST_Centroid(geom) AS fire_point FROM parcels WHERE gid = 462273;
  ```  
  **Response:** This query selects the centroid of the parcel `462273` and defines it as the fire point.  

---

### Step 4: Create a Risk Zone (1 km around the fire point)  

1. **In QGIS:**  
   - Duplicate the `parcel` layer and rename it to `fire-risk`.  
   - Right-click on this layer, then click **Filter...**.  
   - In the **filter expression** area, add the following clause:  
     ```sql
     ST_DWithin(geom, (SELECT ST_Centroid(geom) FROM "public"."parcels" WHERE gid = 462273), 1000)
     ```  
     **Note:** Delete the `SELECT * FROM parcels` part if it's present, as it will cause an error.  

2. **Modify the Symbology:**  
   - Change the symbology of this layer to clearly represent the fire risk zones.  

3. **Add a Second Fire Point:**  
   - Identify another fire point in the parcel `460957`.  
   - Modify the filter to detect zones around both parcels.  

---

### Step 5: Answer Questions with Spatial SQL Queries  

1. **How many parcels are within a 1 km radius of the two target parcels (`gid = 460957` and `gid = 462273`)?**  
   - **SQL Command:**  
     ```sql
     SELECT COUNT(*) 
     FROM parcels 
     WHERE ST_DWithin(geom, (SELECT ST_Union(ST_Centroid(geom)) 
                             FROM parcels 
                             WHERE gid IN (460957, 462273)), 1000);
     ```  
     **Response:** This query returns the total number of parcels within a 1 km radius around the two fire points.  

2. **What is the total area of parcels near the fire?**  
   - **SQL Command:**  
     ```sql
     SELECT SUM(ST_Area(geom)) AS total_area 
     FROM parcels 
     WHERE ST_DWithin(geom, (SELECT ST_Union(ST_Centroid(geom)) 
                             FROM parcels 
                             WHERE gid IN (460957, 462273)), 1000);
     ```  
     **Response:** This query returns the total area of parcels near the two fire points, expressed in the units of the SRID used (square meters for SRID 4326).  

---

## Resources  
- **Official PostGIS Documentation:** [PostGIS Reference](https://postgis.net/docs/reference.html)  

---  