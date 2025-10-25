# Data Dictionary for Horticultural Produce and Market Database 

 This data dictionary desribes the schema of the database tables used in the Horticultural Produce and Markets Database project, including table structures, column definitions, and database constraints to help developers and analysts clearly understand the data model. 

---
 
## Table: producers
Stores information about farms that produce the hortiucltural produce for the markets.
 
<img width="1110" height="122" alt="Image" src="https://github.com/user-attachments/assets/023dcd16-c66a-43dc-a14d-640679d42da5" />
---
 
## Table: products
Contains details about specific farm produce or inventory harvested for sale.

 <img width="1242" height="119" alt="Image" src="https://github.com/user-attachments/assets/31edae9c-3161-445d-9193-3efd24727c48" />
 
---
 
## Table: markets
Lists the various locations where the hortiucltural farm produce is sold.

<img width="1112" height="102" alt="Image" src="https://github.com/user-attachments/assets/23f411ab-5836-4478-83ef-e19824d0e552" />
---
 
 **Notes:**  
- Primary keys uniquely identify rows in each table.
Foreign keys (FK) connect relationships across tables (producers/source → produce/inventory).
This schema supports queries like:
- List all produce sourced from a specific producer
- Find all upcoming produce after a specific season
- Show all produce sold in a given market
- Show producer details
and many more queries based on your criteria. 
