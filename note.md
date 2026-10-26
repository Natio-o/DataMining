# Short Note For Assignment

---

### 4a Customer identifier columns

So there are two columns that look like they identify customers - `customer_id` and `customer_unique_id`. At first glance it seems redundant but they actually serve diffrent purposes.

`customer_id` is basically a surrogate key thats generated fresh for every single order. So if the same person buys something 3 times, they get 3 different `customer_id` values. There are 99,441 of them and every one is unique - which makes sense since its one per order.

`customer_unique_id` on the other hand is the actual identifier for the real person. There are only 96,096 distinct values across the 99,441 rows, which tells us some customers came back and ordered again (I found one person with 17 orders which is pretty active). So if you want to count how many actual customers there are you'd use `COUNT(DISTINCT customer_unique_id)`, not `customer_id` since that would just give you the number of orders.

### 4b Junction table

`order_items` is the junction table here. It sits between orders and products, and also between orders and sellers, resolving what would otherwise be M:N relationships. One order can have multiple products from different sellers, and one product or seller can appear across many orders.

It has a composite primary key of `(order_id, order_item_id)` and holds foreign keys to `orders`, `products`, and `sellers`. This breaks down the many-to-many relationships into managable 1:N links.

### 4c Strictness vs optionality

**Where the data is stricter than the diagram:** The customers to orders relationship. You'd normally draw this as 1:N since logically a customer can place many orders. But because `customer_id` is regenerated per order in this dataset, the data actually shows a strict 1:1 relationship - every `customer_id` maps to exactly one order and vice versa. So the diagram says "many" but the data says "exactly one".

**Where the circle (zero) is justified:** The orders to order_items relationship. I ran an ANTI JOIN and found 775 orders that have zero items attached to them. These are mostly cancelled or unavailable orders (603 unavailable, 164 cancelled, etc). So the circle on the crow's foot is necessary here - not every order has line items, making it 0..N rather than 1..N.
