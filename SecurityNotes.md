
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

CREATE POLICY "Users can select own tickets"
ON tickets FOR SELECT
USING (
  exists (
    select 1 from customers c
    where c.customer_id = tickets.customer_id
      and c.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can insert own tickets"
ON tickets FOR INSERT
WITH CHECK (
  exists (
    select 1 from customers c
    where c.customer_id = tickets.customer_id
      and c.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can update own tickets"
ON tickets FOR UPDATE
USING (
  exists (
    select 1 from customers c
    where c.customer_id = tickets.customer_id
      and c.auth_user_id = auth.uid()
  )
)
WITH CHECK (
  exists (
    select 1 from customers c
    where c.customer_id = tickets.customer_id
      and c.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can delete own tickets"
ON tickets FOR DELETE
USING (
  exists (
    select 1 from customers c
    where c.customer_id = tickets.customer_id
      and c.auth_user_id = auth.uid()
  )
);
```
- payments (users can manage payments for tickets they own; admins full access)
``` sql
CREATE POLICY "Admins full access payments"
ON payments FOR ALL
USING (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'))
WITH CHECK (exists (select 1 from profiles p where p.id = auth.uid() and p.role = 'admin'));

CREATE POLICY "Users can select payments for their tickets"
ON payments FOR SELECT
USING (
  exists (
    select 1 from tickets t
    join customers c on c.customer_id = t.customer_id
    where t.ticket_id = payments.ticket_id
      and c.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can insert payments for their tickets"
ON payments FOR INSERT
WITH CHECK (
  exists (
    select 1 from tickets t
    join customers c on c.customer_id = t.customer_id
    where t.ticket_id = payments.ticket_id
      and c.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can update payments for their tickets"
ON payments FOR UPDATE
USING (
  exists (
    select 1 from tickets t
    join customers c on c.customer_id = t.customer_id
    where t.ticket_id = payments.ticket_id
      and c.auth_user_id = auth.uid()
  )
)
WITH CHECK (
  exists (
    select 1 from tickets t
    join customers c on c.customer_id = t.customer_id
    where t.ticket_id = payments.ticket_id
      and c.auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can delete payments for their tickets"
ON payments FOR DELETE
USING (
  exists (
    select 1 from tickets t
    join customers c on c.customer_id = t.customer_id
    where t.ticket_id = payments.ticket_id
      and c.auth_user_id = auth.uid()
  )
);

```

------------------------------------------------------------------------

## 👨‍💼 Admin Policies

Admins have **full access**.
Use **SECURITY DEFINER** functions with internal role checks as defense-in-depth.

- Delete event (admin only)
``` sql
CREATE OR REPLACE FUNCTION delete_event_by_admin(eid INT)
RETURNS VOID
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  IF NOT EXISTS (SELECT 1 FROM profiles p WHERE p.id = auth.uid() AND p.role = 'admin') THEN
    RAISE EXCEPTION 'Only admins may call delete_event_by_admin';
  END IF;
  DELETE FROM events WHERE event_id = eid;
END;
$$;
```
- Delete ticket (admin only):
  
``` sql
CREATE OR REPLACE FUNCTION delete_ticket_by_admin(tid INT)
RETURNS VOID
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  IF NOT EXISTS (SELECT 1 FROM profiles p WHERE p.id = auth.uid() AND p.role = 'admin') THEN
    RAISE EXCEPTION 'Only admins may call delete_ticket_by_admin';
  END IF;
  DELETE FROM tickets WHERE ticket_id = tid;
END;
$$;

```

------------------------------------------------------------------------

## ⚡️ Testing Roles & Policies

### ✅ User Tests
1.  **SELECT own tickets** (works)\
2.  **INSERT a new ticket purchase** (works)\
3.  **UPDATE or DELETE tickets/payments** (blocked)\
   
- **Create test accounts** in Supabase Auth:
 - Admin user (set profiles.role = 'admin')
 - Regular user A and user B
- **Create/Update profiles** or use signup trigger to auto-create a profile for each auth.users row.. **Populate sample data** (events, customers with auth_user_id, tickets, payments). Ensure customers.auth_user_id points to users.
- Test with **supabase** :
 -Sign in as regular user → request tickets and payments. Confirm user only sees their own.
 - Try to perform admin-only actions (update/delete events) as regular user → **should fail**.
- Sign in as **admin** → confirm full access, call admin RPCs (delete_event_by_admin).
- Use **SQL Editor** for debugging only (SQL Editor runs as service_role and bypasses RLS — do not use this for policy tests).

### ✅ Admin Tests

1.  **UPDATE event details** (works)\
2.  **DELETE event**(works)\
3.  **SELECT all tickets** (works)

------------------------------------------------------------------------

<img width="1903" height="832" alt="image" src="https://github.com/user-attachments/assets/58d58dc6-db66-4607-8d37-d2769784097b" />


## 🛠 Admin-only Function

Example: Admin deletes an event safely.

``` sql
CREATE OR REPLACE FUNCTION delete_event_safe(event_id INT)
RETURNS VOID
LANGUAGE SQL
SECURITY DEFINER
AS $$
  DELETE FROM events WHERE event_id = $1;
$$;

```

------------------------------------------------------------------------

<img width="1901" height="796" alt="image" src="https://github.com/user-attachments/assets/85a3c200-c0d4-408a-a43d-456e601bf6d4" />


## 📎 Reference

-   Linked to [README.md](https://github.com/Evans-dotcom/Data-Tools/blob/Data_Fundamentals-Branch/ReadMe.md)
-   Supabase Policies: <https://supabase.com/docs/guides/database/postgres/row-level-security>
  
