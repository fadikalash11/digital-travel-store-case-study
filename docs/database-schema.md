# Database Schema

The whole catalog lives in a single Supabase (PostgreSQL) table: `public.products`. The storefront, the admin panel, and the n8n delivery workflow all read from it, so adding a row is enough to put a new product on sale.

## `products` table

| Column | Type | Nullable | Purpose |
|---|---|---|---|
| `id` | `uuid` | No | Primary key |
| `title` | `text` | No | Display name; also used by n8n to look the product up after payment |
| `description` | `text` | Yes | Product description on the storefront |
| `price` | `numeric` | No | Price shown to the customer |
| `image_url` | `text` | Yes | Cover image |
| `delivery_type` | `text` | No | `pdf` or `link`, decides how n8n delivers the product |
| `delivery_url` | `text` | No | Public file URL (Supabase Storage) or interactive map URL |
| `email_subject` | `text` | No | Subject of the delivery email |
| `email_body` | `text` | No | Body of the delivery email |
| `is_active` | `boolean` | No | Only active products are shown on the storefront |
| `sort_order` | `integer` | No | Display order |
| `created_at` | `timestamptz` | No | Creation time |
| `updated_at` | `timestamptz` | No | Last update time |
| `features` | `array` | Yes | Bullet-point features shown on the product card |
| `badge` | `text` | Yes | Optional label on the card |

### Columns the n8n workflow depends on

Only five columns are part of the delivery contract:

`title` · `delivery_type` · `delivery_url` · `email_subject` · `email_body`

Everything else exists for the storefront and the admin panel.

## Row Level Security

RLS is enabled on the table with two policies:

| Policy | Command | Role | Rule |
|---|---|---|---|
| Public can view active products | `SELECT` | `public` | `is_active = true` |
| Admin full access on products | `ALL` | `authenticated` | allowed |

In practice:

- Visitors can only read products that are marked active. Deactivating a product hides it immediately, without deleting it.
- Creating, editing, and deleting rows requires a signed-in session, which is what the `/admin` panel provides through Supabase Auth.

## Storage

PDF files are uploaded from the admin panel to a Supabase Storage bucket named `product-files`. The panel stores the generated file URL in `delivery_url`, and n8n downloads the file from that URL at delivery time.

## Why this shape

Moving delivery type and email copy into the table was the key decision. The automation no longer needs to know anything about individual products, and the client can change a product, its price, or even the wording of its delivery email from a form.
