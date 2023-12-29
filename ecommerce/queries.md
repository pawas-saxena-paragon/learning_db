### User stories.

Skipping CRUD queries.

- login user
- create user / signup check what needs to be updated

- read all addresses of user

- query products By - size, brand, price range, color,
- get all variations of product
- get all sizes of product and product variation
- get all promotions valid on product
- get quantity in stock of product , variation , size
- get all products by a promotion
- get new products
- get products that are in stock given variation option and size

- get all orders of a customer
- get all payments of a order
- convert the cart into an order or place an order . update inventory, make payments

- get all products within a category . Here include all subcategories a s well
- get all subcategories of a product

- get cart for user
- add items to cart
- get sum of all items prices

- get all reviews of a product for a given rating
- get all reviews of a user
- get all reviews of a product
- create a review
- get products of a user that dont have a review
- get products by popularity (desc number of reviews)

### Transaction considerations

1. how to prevent simultaneous buying of a product as the qty that we have is limited and there is a delay in the order payment and order creation
2. how to have atomicity between payment and order creation

### Index considerations

1. which queries are going to be fetch most
2. which fields are going to be frequently updated

3. User queries
   a. create user

   ```sql
       insert into user (email, pass_hash, primary_phone )
       values('john@gmail.com', 'giberish', '999999999')
   ```

   b. get all address

   ```sql
       select * from address where user.id = <user_id>
   ```

4. Category
   a. get all children category for a parent category

   ```sql
       select child.id
           from category parent join category child  on child.parent_category_id = parent.id
           where parent.id = <category_id>
   ```

   b. get products within a category

   ```sql
    select product_id from product where category_id in (
        select child.id
           from category parent join category child  on child.parent_category_id = parent.id
           where parent.id = <category_id>
    ) or category_id = <category_id>
   ```

5. Products
   a. get all products

   ```sql
        select * from products limit 50 offset 0
   ```

   b. get all products based on a category and brand and promotion

   ```sql
       select * from products
       where category_id in (
            select child.id
                from category parent join category child  on child.parent_category_id = parent.id
                where parent.id = <category_id>
       )
       and brand_id = <brand_id>
       and promotion_id = <promotion_id>
   ```

   c. get all products for a particular price range sorting them by price

   ```sql
    select product_id
        from products p join product_sk ps on p.product_id = ps.product_id
        where ps.price between <start_price> and <end_price>
        order by ps.price
   ```

   d. get all products of a particular size

   ```sql
    select product_id
        from products p join product_sk ps on p.product_id = ps.product_id
        where ps.size = <size_id>
   ```

   e. get all variation types of a product

   ```sql
    select vot.variation_id, vot.name
        from
            products p join product_sku ps on p.product_id = ps.product_id
            join variation_option vo  on vo.variation_option_id = ps.variation_option_id
            join varation_option_type vot on vo.variation_id = vot.variation_id
        where p.product_id = <product_id>
   ```

   f.get all product variations options for a given product . Use case would be to show the product along with all variation options

   ```sql
     select p.product_id, p.brand_id ,ps.sku, ps.price, ps.product_sku_image , vo.variation_option_id, vo.name, vo.variation_id
        from
            products p join product_sku ps on p.product_id = ps.product_id
            join variation_option vo  on vo.variation_option_id = ps.variation_option_id
        where p.product_id = <product_id>
   ```

   g. get all available sizes for a product variation option id

   ```sql
      select ps.size_id
        from
            products p join product_sku ps on p.product_id = ps.product_id
            join variation_option vo  on vo.variation_option_id = ps.variation_option_id
        where p.product_id = <product_id> and vo.variation_option_id = <variation_option_id>
   ```

   h. get all products for a variation option and size that are in stock

   ```sql
       select * from
           product p join product_sku ps on p.product_id = ps.product_id
           where ps.size_id = <size_id> and ps.variation_option_id = <variation_option_id>
           and units_in_stock > 0
   ```

   i. get all new products - this is basically going to happen along with one of the previously mentioned queries so just add a
   sort by `created_at desc` at the end of query

6. Cart
   a. get cart for a user

   ```sql
        select * from cart where user_id = <user_id>
   ```

   b. get all cart items for a user

   ```sql
    select ci.sku, ci.qty from cart c join cart_item ci on c.cart_id = ci.cart_id
    where c.user_id = <user_id>
   ```

   c. get sum of all items prices in cart for a user. todo add promotion discount

   ```sql
    select sum(ps.price * ci.qty)  total
        from cart c join cart_item ci  on c.cart_id = ci.cart_id
        join product_sku ps on ps.sku = ci.sku
        where c.user_id = <user_id>
        -- no grouping required here as doing it for the entire result. user_id grouping would have made sense otherwise
   ```

   d. add / delete items to cart - just insert values in `cart_item` table with the item `sku` or update the `qty` if applicable for a given
   cart it.

7. Orders
   a. convert cart into orders. these things happen - inventory is reduced, payment is created,  order is created .
   items are deleted from cart and moved to sku order table. 
   consider discount also
   ```sql

   ```