## Aggio SQL Assessment

#### Let's suppose you have the following tables in PostgreSQL.

```SQL
create table customers (
    id int primary key not null,
    first text not null,
    last text not null
);

create table products (
      id int primary key not null,
      product text not null,
      price_dollars int not null
);

create table sales (
      id int primary key not null,
      customer_id int references customers(id) not null,
      product_id int references products(id) not null,
      units int not null
);

insert into customers
    (id, first, last)
values
    (1, 'Dale', 'Gribble'),
    (2, 'Rusty', 'Shackleford'),
    (3, 'Mona', 'Lisa');

insert into products
    (id, product, price_dollars)
values
    (1, 'Cheeseburger', 10),
    (2, 'Hamburger', 9),
    (3, 'Hotdog', 5);

insert into sales
    (id, customer_id, product_id, units)
values
    (1, 2, 2, 2),
    (2, 1, 1, 1);
```
<br />

#### Using the tables above write a query to extract the customers that had no sales.


```SQL
select c.id, c.first, c.last
from customers c
left join sales s on s.customer_id = c.id
where s.customer_id is null;
```
<br />

#### Using the tables above write a query to display - customer name, product and the price they paid. (Notice that customer_id = 2 bought 2 units. Include a group by.)

```SQL
select
    --r.sale_id,
    --r.customer_id,
    r.customer_name,
    --r.product_id,
    r.product_name,
    r.unit_price,
    r.units_ordered,
    r.total_price_paid
from (
    select
        s.id as sale_id,
        s.customer_id,
        concat(c.first, ' ', c.last) as customer_name,
        s.product_id,
        p.product as product_name,
        sum(p.price_dollars) as unit_price,
        sum(s.units) as units_ordered,
        sum(p.price_dollars * s.units) as total_price_paid
    from customers c
        inner join sales s
            on s.customer_id = c.id
        inner join products p
            on p.id = s.product_id
    group by 1, 2, 3, 4, 5
    order by 2
) r;

/*
Comment on the above solution. The outer query is not necessary.
I've included it so that if we need to insert fields in the future,
we can uncomment the fields and there they will be. Otherwise, you
have to change the numbering in group by.
*/
```
<br />

#### You are tasked with creating an etl_log table that will capture files coming into the server. Create a table schema to track and store the attributes of the files (be sure to include datatypes and write it for postgresql or mysql but specify which).

```SQL
/*
First, as I've noted above, I'm using postgres. Here I see that
I'm not told much about the etl_log table and I will have to think
of all of the common attributes that files may have that may be
important. I will describe my thinking process here using assumptions.

We're dealing with files, possibly at an ingestion point. We would
probably want to know the following attributes of the incoming files:

    - file_id
    - file_hash
    - file_name
    - image_flag
    - file_size
    - file_origin
    - binary_flag
    - received_datetime

We could possibly have many other attributes. It all depends on what
the need is and the purpose of the log.

Here is how I would create this table.
*/

create table etl_log (
    file_id int primary key not null,
    file_hash text not null,
    file_name text not null,
    image_flag boolean not null,
    file_size int not null,
    file_origin text not null,
    binary_flag boolean not null,
    received_datetime timestamp not null
);
```
<br />

#### Let's suppose you have the following table.

```SQL
create table monthly_location_sales (
      location_id int primary key not null,
      jan int,
      feb int,
      mar int,
      apr int,
      may int,
      jun int,
      jul int,
      aug int,
      sep int,
      oct int,
      nov int,
      dec int
);

insert into monthly_location_sales
            (location_id, jan, feb, mar, apr, may, jun, jul, aug, sep, oct, nov, dec)
    values
            (1, 1, 2, 1, 5,	3, 8, 1, 6,	8, 4, 96, 7),
            (2,	3, 1, 23, 9, 1, null, 2, 47, 1, 9, 2, 3),
            (3,	1, 3, 5, null, 5, 4, 9, 33, 2, 8, 2, 4);
```
<br />

#### Pivot this table to result in a table with the columns: location_id, Month, Value (i.e. take it from "wide" to "long") using whatever method you’d like. Be sure not to create a Pivot Table out of the data and that your solution would work if there were 100 columns without hard-coding column names. Hint - python makes this easy.

```SQL
select mls.location_id, mv.month, mv.value
from monthly_location_sales mls
    cross join lateral
        jsonb_each_text(to_jsonb(mls) - 'location_id') as mv(month, value);
```
