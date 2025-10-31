# Horticulture Data Project

<div align="center">
  <img width="200" height="200" alt="Horticulture Logo" src="https://github.com/user-attachments/assets/20661293-a214-4004-9042-657102fb0710" />
  <br/>
  <h2><b>Horticulture Project — Producers • Products • Markets</b></h2>
</div>

---

## 📗 Table of Contents

- [About the Project](#about-the-project)  
- [Built With](#built-with)  
- [Key Features](#key-features)  
- [Getting Started](#getting-started)  
  - [Prerequisites](#prerequisites)  
  - [Setup](#setup)  
  - [Usage](#usage)  
  - [Connecting from Posit to Supabase](#connecting-from-posit-to-supabase)  
- [Schema SQL](#schema-sql)  
- [R Data Analysis](#r-data-analysis)  
- [Data Dictionary](#data-dictionary)  
- [Authors](#authors)  
- [Future Features](#future-features)  
- [Contributing](#contributing)  
- [Support](#support)  
- [Acknowledgements](#acknowledgements)  
- [FAQ](#faq)  
- [License](#license)

---

## 📖 About the Project

This repository models a small horticulture supply-chain database: **farm producers**, **products** (produce listings), and **markets**.  
It includes sample data and an R workflow for connecting to the database, exploring the data, and building visualizations to inform pricing, seasonality, and market coverage.

Use this project to learn:
- designing a PostgreSQL schema for horticulture data,
- seeding data in Supabase,
- connecting Posit (RStudio) to run queries and create exploratory visualizations.

---

## 🛠 Built With

- **Database & Hosting:** Supabase (PostgreSQL)  
- **SQL:** Schema creation + sample data inserts (`schema.sql`)  
- **R / Posit:** DBI, RPostgres, dplyr, ggplot2, plotly, DT

---

## Key Features

- Relational schema linking producers → products (with cascade deletes).  
- Sample data for producers, products (harvest season + price), and markets.  
- R script for connecting to Supabase from Posit and producing interactive visualizations.  
- Quick analyses: price-by-product, price-by-season, producers by location, markets by open day.

---

## 💻 Getting Started

### Prerequisites

- Supabase account & project (PostgreSQL)  
- Posit / RStudio (Posit Cloud or local)  
- R packages: `DBI`, `RPostgres`, `dplyr`, `ggplot2`, `plotly`, `DT`

### Setup

Clone the repository:
```bash
git clone https://github.com/your-username/horticulture-db.git
cd horticulture-db
```

1. Open your Supabase project.
2. Go to **SQL Editor** → create a new query → paste the **Schema SQL** below and run it.
3. Note the database connection details: host, dbname, user, password, port (found in Project → Settings → Database → Connection info).

### Usage

1. In Posit, create a file `connect_db.R` containing a `connect_db()` function (example below).  
2. Run the provided R script `visualize_supabase_tables.R` to fetch tables and render interactive visualizations.  
3. Modify queries/visualizations as needed.

---

## Connecting from Posit to Supabase

Install packages:
```r
install.packages(c("DBI", "RPostgres", "dplyr", "ggplot2", "plotly", "DT"))
```

Example `connect_db.R`:
```r
library(DBI)
connect_db <- function() {
  DBI::dbConnect(
    RPostgres::Postgres(),
    dbname = Sys.getenv("DB_NAME", "postgres"),
    host   = Sys.getenv("DB_HOST", "your-project.supabase.co"),
    port   = as.integer(Sys.getenv("DB_PORT", 5432)),
    user   = Sys.getenv("DB_USER", "postgres"),
    password = Sys.getenv("DB_PASS"),
    sslmode = "require"
  )
}
```

Tip: In Posit Cloud set `DB_HOST`, `DB_USER`, `DB_PASS`, `DB_NAME`, `DB_PORT` in Project → Tools → Environment.

---

## 💾 Schema SQL

Run the following SQL in Supabase SQL editor (`schema.sql`):

```sql
-- Drop old tables if they exist
DROP TABLE IF EXISTS producers CASCADE;
DROP TABLE IF EXISTS products CASCADE;
DROP TABLE IF EXISTS markets CASCADE;

-- create farm producers table
CREATE TABLE public.producers (
  id serial primary key,
  farm_name varchar(100) not null,
  location varchar(100) not null,
  produce_type varchar(50) not null,
  contact_email varchar(100) not null
);

-- Create product catalog table
CREATE TABLE public.products (
  id serial primary key,
  product_name varchar(100) not null,
  variety varchar(100),
  harvest_season varchar(50) not null,
  price_per_unit numeric(6,2) not null,
  producer_id int references public.producers(id) on delete cascade
);

-- Create markets table
CREATE TABLE public.markets (
  market_id serial primary key,
  market_name varchar(100) not null,
  city varchar(100) not null,
  is_farmers_market boolean not null default true,
  open_day varchar(20)
);

-- Insert farm producers (5 rows)
INSERT INTO public.producers (farm_name, location, produce_type, contact_email) VALUES
  ('Freshgold Kenya', 'Nakuru, Kenya', 'Vegetables', 'freshgoldke@email.com'),
  ('Lenara Belle', 'Limuru, Kenya', 'Fruits', 'fruits@lenarablle.co.ke'),
  ('Signum Fresh', 'Kisumu, Kenya', 'Leafy Greens', 'signumf@production.io'),
  ('Alfa Enterprises', 'Nairobi, Kenya', 'Berries', 'operations@alfaent.co.ke'),
  ('Diakim International', 'Garissa, Kenya', 'Herbs', 'herbspices@diakim.com');

-- Insert produce listings (5 rows)
INSERT INTO public.products (product_name, variety, harvest_season, price_per_unit, producer_id) VALUES
  ('Onion', 'Bombay Red', 'Cool Dry', 3.59, 1),
  ('Passionfruit', 'Purple', 'Short Rains', 6.35, 2),
  ('Cabbage', 'Sugarloaf', 'Long Rains', 3.00, 3),
  ('Strawberry', 'Wendy', 'Cool Dry', 4.99, 4),
  ('Basil', 'Sweet', 'Long Rains', 2.45, 5);

-- Insert markets (5 rows)
INSERT INTO public.markets (market_name, city, is_farmers_market, open_day) VALUES
  ('City Center Market', 'Busia, Kenya', TRUE, 'Saturday'),
  ('Makadara Market', 'Thika, Kenya', FALSE, 'Daily'),
  ('Organic Farmers Market', 'Nairobi, Kenya', FALSE, 'Monday'),
  ('Kimbo Market', 'Nyeri, Kenya', TRUE, NULL),
  ('Kinangop Traders Market', 'Naivasha, Kenya', TRUE, 'Thursday');
```

---

## 📊 R Data Analysis

Compact R workflow to fetch the three tables and create interactive plots. Save as `visualize_supabase_tables.R` and run after `connect_db()` is configured.

```r
# visualize_supabase_tables.R (compact)
library(DBI); library(RPostgres); library(dplyr); library(ggplot2); library(plotly); library(DT)

con <- connect_db()

producers <- dbGetQuery(con, "SELECT * FROM public.producers ORDER BY id")
products  <- dbGetQuery(con, "SELECT * FROM public.products ORDER BY id")
markets   <- dbGetQuery(con, "SELECT * FROM public.markets ORDER BY market_id")

# Interactive table preview
DT::datatable(producers, caption = "Producers")
DT::datatable(products, caption = "Products")
DT::datatable(markets, caption = "Markets")

# Join products to producers
prod_with_producer <- products %>%
  left_join(producers %>% select(id, farm_name, location, produce_type),
            by = c("producer_id" = "id"))

# Price per product (interactive)
p1 <- ggplot(prod_with_producer, aes(x = reorder(product_name, price_per_unit), y = price_per_unit,
             text = paste("Variety:", variety, "<br>Producer:", farm_name))) +
  geom_col() + coord_flip() + labs(title="Price per unit by product", x="", y="Price") + theme_minimal()
print(ggplotly(p1, tooltip = "text"))

# Price by harvest season (box + points)
p2 <- ggplot(prod_with_producer, aes(x = harvest_season, y = price_per_unit,
             text = paste(product_name, "<br>Price:", price_per_unit))) +
  geom_boxplot(outlier.shape = NA) + geom_jitter(width=0.2) +
  labs(title="Price distribution by harvest season", x="Harvest season", y="Price")
print(ggplotly(p2, tooltip = "text"))

dbDisconnect(con)
```

---

## Output in R Posit Cloud
**Outputs for database connection in Posit**
<img width="1357" height="485" alt="image" src="https://github.com/user-attachments/assets/962d94f5-cd2c-492c-95e6-e059755421cc" />

**Output for Producers**
<img width="1116" height="443" alt="image" src="https://github.com/user-attachments/assets/f4fd296c-643b-43af-80b5-375b2fa941da" />

**Output for Products**
<img width="1365" height="474" alt="image" src="https://github.com/user-attachments/assets/b166886a-b713-4b54-8aa1-5667438604ca" />

## Visualization in Posit

**price per unit by product**

```R library(dplyr)
# Join products to producers
prod_with_producer <- produce %>%
  left_join(producers %>% select(id, farm_name, location, produce_type),
            by = c("producer_id" = "id"))
library (ggplot2)
# Price per product (interactive)
p1 <- ggplot(prod_with_producer, aes(x = reorder(product_name, price_per_unit), y = price_per_unit,
                                     text = paste("Variety:", variety, "<br>Producer:", farm_name))) +
  geom_col() + coord_flip() + labs(title="Price per unit by product", x="", y="Price") + theme_minimal()
print(ggplotly(p1, tooltip = "text"))
# Join products to producers
prod_with_producer <- products %>%
  left_join(producers %>% select(id, farm_name, location, produce_type),
            by = c("producer_id" = "id"))
```
<img width="1361" height="593" alt="image" src="https://github.com/user-attachments/assets/efe88d2c-95ae-45cb-aa8b-da9c6628a37e" />

**Price distribution by season**
```R
p2 <- ggplot(prod_with_producer, aes(x = harvest_season, y = price_per_unit,
                                     text = paste(product_name, "<br>Price:", price_per_unit))) +
  geom_boxplot(outlier.shape = NA) + geom_jitter(width=0.2) +
  labs(title="Price distribution by harvest season", x="Harvest season", y="Price")
print(ggplotly(p2, tooltip = "text"))
```
<img width="1354" height="552" alt="image" src="https://github.com/user-attachments/assets/59c3b99b-2354-4d15-9151-dedb2722ade9" />






## Data Dictionary

**public.producers**
- `id` — primary key (serial)  
- `farm_name` — producer farm name (varchar)  
- `location` — city/region (varchar)  
- `produce_type` — vegetables/fruits/herbs/berries (varchar)  
- `contact_email` — contact email (varchar)  

**public.products**
- `id` — primary key  
- `product_name` — e.g., Onion, Basil  
- `variety` — variety name  
- `harvest_season` — season label (e.g., Long Rains, Cool Dry)  
- `price_per_unit` — numeric price  
- `producer_id` — FK → producers(id)  

**public.markets**
- `market_id` — primary key  
- `market_name` — name of market  
- `city` — market location  
- `is_farmers_market` — boolean  
- `open_day` — day(s) market opens (nullable)  

---

## Authors

**Project Owner / Maintainer**  
- Your Name Here (replace with your name)  
- GitHub: @your-github  
- Contact: your.email@example.com

---

## Future Features

- Expand schema: shipments, buyers, inventory, and transactions.  
- Add time-series pricing and seasonal trend analysis.  
- Small Shiny dashboard combining interactive maps, price trends, and producer filters.  
- Automate daily price ingestion from CSV uploads or API.

---

## Contributing

Contributions are welcome. Please:
1. Fork the repo.  
2. Create a feature branch (`git checkout -b feature/your-feature`).  
3. Commit your changes and open a Pull Request with a clear description.

---

## Support

If you find this project useful, give it a ⭐️ on GitHub and share feedback via Issues.

---

## Acknowledgements

- [Supabase](https://supabase.com) — PostgreSQL hosting  
- [Posit](https://posit.co) — RStudio / Posit Cloud environment for analysis

---

## FAQ

**1. How do I run the schema?**  
Open Supabase SQL Editor and paste the `Schema SQL` section; run it to create tables and seed data.

**2. I get `Tenant or user not found` when connecting — why?**  
That means your DB credentials (host, user, password) are incorrect or don’t match the Supabase project. Double-check Project → Settings → Database → Connection info and use the database **user/password** (not API keys) in `connect_db()`.

**3. Can I add more sample data?**  
Yes — append `INSERT` statements or upload CSVs into Supabase table editor.

**4. Can I use this with another DB (MySQL)?**  
This schema and the provided R code are written for PostgreSQL (Supabase). Converting to MySQL will require datatype and connection adjustments.

---

## License

This project is released under the **MIT License**. See `LICENSE` for details.
