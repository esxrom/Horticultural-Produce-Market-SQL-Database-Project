# Horticultural-Produce-Market-SQL-Database-Project
A relational database schema for tracking produce, producers and markets in Kenya.

<a name="readme-top"></a>

<!-- TABLE OF CONTENTS -->

# 📗 Table of Contents

- [My SQL Project](#about-project)
- [📗 Table of Contents](#-table-of-contents)
- [📖 My SQL Project](#about-project)
  - [🛠 Built With ](#-built-with-)
    - [Tech Stack ](#tech-stack-)
    - [Key Features ](#key-features-)
  - [💻 Getting Started ](#-getting-started-)
    - [Prerequisites](#prerequisites)
    - [Setup](#setup)
    - [Usage](#usage)
  - [👥 Authors ](#-authors-)
  - [🔭 Future Features ](#-future-features-)
  - [🤝 Contributing ](#-contributing-)

<!-- PROJECT DESCRIPTION -->

# 📖 My SQL Project <a name="about-project"></a>

**My SQL Project** is a simple Database that uses SQL, Postgres via Supabase to create, query and secure a **Horticulture Produce, Producers, and Markets** database.

## 🛠 Built With <a name="built-with"></a>

### Tech Stack <a name="tech-stack"></a>
- SQL
- Postgres DB

<!-- Features -->

### Key Features <a name="key-features"></a>

- [ ] **Tables**
- [ ] **Schema**
- [ ] **Access control**

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- GETTING STARTED -->

## 💻 Getting Started <a name="getting-started"></a>

To rebuild this DB, follow these steps.

### Prerequisites

To run this project, you need:
- [A Supabase account](https://supabase.com/)
- [Knowledge on SQL](https://www.w3schools.com/sql/)
- A schema for creating your tables in the DB

<!-- ### Setup -->
### Setup

Copy the contents of this Readme.md to your Project's file

OR

Clone this repository to your desired folder:

```sh
  git clone [https://github.com/esxrom/Horticultural-Produce-Market-SQL-Database-Project]
```

<!-- ### DB Creation -->

### DB Schema

- The DB is made up of 3 tables. Each table has 5 entries.
- To create the table, you will need a schema as shown below:

```sql
-- Drop old tables if they exist
DROP TABLE IF EXISTS producers CASCADE;
DROP TABLE IF EXISTS product CASCADE;
DROP TABLE IF EXISTS markets CASCADE;

-- create farm producers table
create table public.producers (
id serial primary key,
farm_name varchar (100) not null,
location varchar (100) not null,
produce_type varchar (50) not null,
contact_email varchar (100) not null
);

-- Create product catalog table
create table public.products (
  id serial primary key,
  product_name varchar (100) not null,
  variety varchar (100),
  harvest_season varchar (50) not null,
  price_per_unit numeric (6, 2) not null,
  producer_id int references public.producers(id) on delete cascade
);

-- Create markets table
create table public.markets (
market_id serial primary key,
market_name varchar (100) not null,
city varchar (100) not null,
is_farmers_market boolean not null default true,
open_day varchar (20)
);

-- Insert farm producers (5 rows)
insert into public.producers (farm_name, location, produce_type, contact_email) values
  ('Freshgold Kenya', 'Nakuru, Kenya', 'Vegetables', 'freshgoldke@email.com'),
  ('Lenara Belle', 'Limuru, Kenya', 'Fruits', 'fruits@lenarablle.co.ke'),
  ('Signum Fresh', 'Kisumu, Kenya', 'Leafy Greens', 'signumf@production.io'),
  ('Alfa Enterprises', 'Nairobi, Kenya', 'Berries', 'operations@alfaent.co.ke'),
  ('Diakim International', 'Garissa, Kenya', 'Herbs', 'herbspices@diakim.com');

-- Insert produce listings (5 rows)
insert into public.products (product_name, variety, harvest_season, price_per_unit, producer_id) values
  ('Onion', 'Bombay Red', 'Cool Dry', 3.59, 1),
  ('Passionfruit', 'Purple', 'Short Rains', 6.35, 2),
  ('Cabbage', 'Sugarloaf', 'Long Rains', 3.00, 3),
  ('Strawberry', 'Wendy', 'Cool Dry', 4.99, 4),
  ('Basil', 'Sweet', 'Long Rains', 2.45, 5);

-- Insert markets (5 rows)
insert into public.markets (market_name, city, is_farmers_market, open_day) values
  ('City Center Market', 'Busia, Kenya', TRUE, 'Saturday'),
  ('Makadara Market', 'Thika, Kenya', FALSE, 'Daily'),
  ('Organic Farmers Market', 'Nairobi, Kenya', FALSE, 'Monday'),
  ('Kimbo Market', 'Nyeri, Kenya', TRUE, NULL),
  ('Kinangop Traders Market', 'Naivasha, Kenya', TRUE, 'Thursday');

```

- The Tables should look like this in Supabase:

producers:
<img width="1348" height="337" alt="Image" src="https://github.com/user-attachments/assets/ccabaf90-e436-4d09-8295-789be85396d8" />

products:
<img width="1341" height="494" alt="Image" src="https://github.com/user-attachments/assets/0d5c8885-0520-4a38-b03d-b71eceb13a27" />

markets:
<img width="1334" height="342" alt="Image" src="https://github.com/user-attachments/assets/dd042093-e8d1-4b9f-9795-9eeb1cbab417" />

- The ERD screenshot from Supabase looks like this: 
<img width="719" height="447" alt="Image" src="https://github.com/user-attachments/assets/cf257dfc-a26e-4920-b3ee-3d03fea9195c" />

- To test the table, I used two queries: 

```sql
select
p.product_name,
p.price_per_unit,
pr.farm_name,
pr.produce_type,
pr.location
from
public.products p
join
public.producers pr on p.producer_id = pr.id
order by
pr.farm_name;
````

```sql
select
p.product_name,
p.price_per_unit,
pr.farm_name,
pr.produce_type
from
public.products p
join
public.producers pr on p.producer_id = pr.id
where
pr.produce_type in ('Herbs', 'Berries')
order by
p.price_per_unit asc;
````

- Here are the results of the queries:
<img width="1353" height="596" alt="Image" src="https://github.com/user-attachments/assets/2dc5c50d-e1e4-4986-a39d-896807f517b3" />


<img width="1343" height="505" alt="Image" src="https://github.com/user-attachments/assets/d532531a-73c1-42f4-8358-cd69c181ae1e" />

<!-- AUTHORS -->

## 👥 Authors <a name="authors"></a>

👤 **Antony Esirom**

- GitHub: [@esxrom](https://github.com/esxrom)
- LinkedIn: [@emadauesirom](https://www.linkedin.com/in/emadau-esirom-1b8335369/)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- FUTURE FEATURES -->

## 🔭 Future Features <a name="future-features"></a>

- [ ] **Add security**
- [ ] **Link DB to R for visualisation purposes and further analyses**

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- CONTRIBUTING -->

## 🤝 Contributing <a name="contributing"></a>

Contributions, issues, and feature requests are welcome!

Feel free to check the [issues page](../../issues/).

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- SUPPORT -->
