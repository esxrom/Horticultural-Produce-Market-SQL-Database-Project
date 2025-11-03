
# Security Notes — Horticulture Database Project | Data Fundamentals

This document explains the **security setup**, **Row Level Security (RLS)**, **roles**, and **policies** for the Horticulture Production Database implemented on Supabase (Postgres). It describes how authentication (Supabase Auth) ties to data, how Admin and User roles are enforced, admin-only functions, and how to test everything step-by-step.

------------------------------------------------------------------------
## Identity & Role Mapping

We use Supabase Auth for user authentication and a small role store to determine admin privileges.
There are two primary database roles to create - **admin_role** & **user_role**.
``` sql
create role admin_role nologin;
create role user_role nologin;
```

The database's built-in **current_role** is used for authentication and to determine identity and privileges of the connected session.
The producers table includes a new column, **owner_role** (VARCHAR), which directly stores the name of the role (e.g., 'user_role') authorized to manage that specific farm's data.
# Example schema snippets (role & linkage):
``` sql
Alter table public.producers
Add column user_role varchar (100);
```

We shall update the existing data, assigning the first 2 producer ids (1, 2) to be 'owned' by a specific user role, the remaining producers (3, 4, 5) will have NULL ownership, meaning only admins can manage those.
``` sql
update public.producers
set owner_role = 'user_role'
where id in (1, 2);

update public.producers
set owner_role = null
where id in (3, 4, 5);
```

# Enable Row Level Security (RLS)

Enable RLS on every table you want protected. RLS denies access to the table by default until we are done with creating all the policies.
``` sql
ALTER TABLE producers ENABLE ROW LEVEL SECURITY;
ALTER TABLE products ENABLE ROW LEVEL SECURITY;
ALTER TABLE markets ENABLE ROW LEVEL SECURITY;

```

***The user roles generated appear as follows in the producers table below***

```sql
SELECT * FROM public.producers;

```
<img width="1920" height="602" alt="image" src="https://github.com/user-attachments/assets/db5cea9f-68a9-4e9d-9aae-ad98ff9db65e" />


------------------------------------------------------------------------

#RLS Policies: Admin vs User Access Roles
## ```Producers``` Table Policies

Admins have full access (ALL) and the rule or policy condition remains TRUE (always allowed).

``` sql
CREATE POLICY admin_full_access_producers ON public.producers
FOR ALL
TO admin_role
USING (TRUE) 
WITH CHECK (TRUE);
```

Normal users have restricted access. They can only READ ALL and manage (update, insert, delete) their OWN rows/records.

``` sql
-- Read all and manage their own rows
create policy user_manage_own_producers on public.producers
for all
to user_role
using (owner_role = current_role::text)
with check (owner_role = current_role::text);

-- User Read-All: a separate select policy is needed for read-all access for producers
create policy user_read_all_producers on public.producers
for select
to user_role
using (true);
```

## ```Products``` Table Policies
Users manage their own product row; admins full access.
``` sql
---Admin Policy (Full Access)
create policy admin_full_access_products on public.products
for all
to admin_role
using (true)
with check (true);

---Users can manage their own products
create policy user_manage_own_products on public.products
for all
to user_role
using (producer_id in (
  select id from public.producers where owner_role = current_role::text
))
with check (producer_id in (
  select id from public.producers where owner_role = current_role::text
));

---Users have read-all access to the products table
create policy user_read_all_products on public.products
for select
to user_role
using (true);
```

## ```Markets``` Table Policies
Admins have full access; users have read only access

``` sql
  create policy admin_full_access_markets on public.markets
for all
to admin_role
using (true)
with check (true);

create policy user_read_markets on public.markets
for select
to user_role
using (true);
```

------------------------------------------------------------------------

## Admin Policies

Admins have **full access**.
Use **SECURITY DEFINER** functions with internal role checks as defense-in-depth.

- Delete product using its id (admin only)
``` sql
create or replace function delete_product_by_admin (product_id_to_delete int)
returns void
language plpgsql
security definer
AS $$
begin
if not exists (
select 1
from pg_roles
where rolname = 'admin_role'
and pg_has_role (current_user, 'admin_role', 'member')
) then
raise exception 'Permission Denied: Only the admin_role may call delete_product_by_admin';
end if;
delete from public.products
where id = product_id_to_delete;
end;
$$;
```
- Delete farm producer using their id (admin only):
  
``` sql
create or replace function delete_producer_by_admin (producer_id_to_delete int)
returns void  
language plpgsql
security definer
as $$
begin
if not exists (
  select 1
  from pg_roles
  where rolname = 'admin_role'
  and pg_has_role (current_user, 'admin_role', 'member')
) then
raise exception 'Permission Denied: Only the admin_role may call delete_producer_by_admin';
end if;
delete from public.producers
where id = producer_id_to_delete;
end;
$$;

```
- The necessary final step to grant execution permission for the admin roles to call the function

``` sql
grant execute on function delete_product_by_admin (int) to admin_role;
grant execute on function delete_producer_by_admin (int) to admin_role;
```

------------------------------------------------------------------------

## ⚡️ Testing Roles & Policies

### ✅ User Tests
The goal is to confirm the user can read all data but can only modify rows where ``` owner_role = current_role.```
1.  **SELECT all products** (works)
2.  **READ all from markets** (works)
3.  **INSERT own product (product_name, producer_name, harvest_season, price_per_unit)** (works)
4.  **UPDATE or DELETE unowned producer or product** (blocked)

### ✅ Admin Tests

1.  **UPDATE unowned producers** (works)
2.  **DELETE farm/producers**(works)
3.  **SELECT all markets** (works)

------------------------------------------------------------------------

<img width="1314" height="591" alt="image" src="https://github.com/user-attachments/assets/519b7630-a442-4131-9e95-ad7c641fb376" />

<img width="998" height="297" alt="image" src="https://github.com/user-attachments/assets/dded2e96-3c4a-4a9f-b513-bc4f479c3c36" />




## 🛠 Admin-only Function

Example: Admin deletes a product safely.

``` sql
select delete_product_by_admin (
 (select id from public.products where product_name = 'Strawberry' limit 1) 
);

```

------------------------------------------------------------------------

<img width="1362" height="349" alt="image" src="https://github.com/user-attachments/assets/982bc25b-e395-48fb-9440-47c064773068" />


## 📎 Reference

-   Linked to [README.md](https://github.com/esxrom/Horticultural-Produce-Market-SQL-Database-Project/blob/tonyesxrom-patch-1/README.md)
-   Supabase Policies: <https://supabase.com/docs/guides/database/postgres/row-level-security>
  
