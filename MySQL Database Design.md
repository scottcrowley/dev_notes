# MySQL Database Design

### Notes on [MySQL Database Design](https://laracasts.com/series/mysql-database-design) from Laracasts and other Miscellaneous MySQL notes

* ## Sakila Example database
    * Throughout the serier, JW uses the [Sakila example database](https://dev.mysql.com/doc/sakila/en/) as a reference. Download the zip file from https://downloads.mysql.com/docs/sakila-db.zip
* ## Foreign Keys
    * A foreign key constraint can be added after a table has already been created by using
        ```mysql
        alter table comments
        add foreign key (post_id) 
        references posts(id)
        on delete cascade
        on update cascade
        ```
    * The default `on` action is `NO ACTION`, which restricts deleting or updating the id of a row that has a foreign key constraint.
    * `on` Actions
        * `cascade` - cascade down and either delete or update all rows that have a matching foreign key constraint
        * `no action` - restricts any updates to id's or the deleting of the row if there is a matching foreign key constraint. Default action.
        * `restrict` - pretty much the same as `no action`.
        * `set null` - sets the foreign key to `null` on a `delete` action.
    * Foreign key constraints in a Laravel migration
        ```php
        $table->foreign('post_id')
            ->references('id')
            ->on('posts')
            ->onDelete('cascade')
        ```
* ## Joins
    * `join` - Joins an additional table based on a foreign key and returns results only when the key has a corresponding match in the joined table
        ```mysql
        select *
        from store
        join address
        on store.address_id = address.address_id
        ```
    * `inner join` - same as `join`
    * `left join` - returns all rows from the left table (store) and will provide details from the right table (address) if there is a match on the key provided
    * `left outer join` - same as `left join`
    * `right join` - returns all rows from the right table (address) and will provide details from the left table (store) if there is a match on the key provided
    * `right outer join` - same as `right join`
* ## Aggregates and Group By
    * An aggregate is when something is done to a particular column like returning the min, max, count, sum, etc.
    * An aggregate requires a `group by` clause
    * A `group by` clause requires an aggregate.
    * To alias an aggregated column just type the alias after the aggregate function without using an `as`
        ```mysql
        count(rentals.rental_id) rentals_checked_out
        ```
    * Example from series using Sakila DB
        ```mysql
        select
            customer.customer_id, 
            customer.first_name, 
            customer.last_name, 
            count(rental.rental_id) rentals_checked_out
        from customer
        left join rental
        on rental.customer_id = customer.customer_id
        group by customer.customer_id
        ```
    * Multiple columns can be used in a `group by` clause.
        ```mysql
        group by customer.customer_id, rentals.rental_id
        ```
* ## Sub Queries and Multiple Joins
    * Sub queries are more time intensive then performing multiple joins. Might be better to use multiple joins in some cases instead of a sub query
    * You can use a sub query in place of any field or table reference. The following sql takes about 110 ms to run.
        ```mysql
        select
            customer.customer_id, 
            customer.first_name, 
            customer.last_name, 
            count(rental.rental_id) rentals_checked_out,
            address.address store_address
        from customer
        left join rental
        on rental.customer_id = customer.customer_id
        left join address
        on address.address_id = (
            select address_id from store where store.store_id = customer.store_id
        )
        group by customer.customer_id, address.address
        ```
    * Same query as above except using multiple joins instead of a sub query. The following sql takes about 50 ms to run.
        ```mysql
        select
            customer.customer_id, 
            customer.first_name, 
            customer.last_name, 
            count(rental.rental_id) rentals_checked_out,
            address.address store_address
        from customer
        left join rental
        on rental.customer_id = customer.customer_id
        left join store
        on store.store_id = customer.store_id
        left join address
        on address.address_id = store.address_id
        group by customer.customer_id, address.address
        ```
* ## Miscellaneous
    * Column aliases can be assigned with or without the `as`
        ```mysql
        address.address store_address
        address.address as store_address
        ```
    * To alias a table, make sure to update all occurenences of the full table name or an error will be thrown. Example: Change the `customer` table name to `c`.
        ```mysql
        select
            c.customer_id, 
            c.first_name, 
            c.last_name, 
            count(rental.rental_id) rentals_checked_out,
            address.address store_address
        from customer c
        left join rental
        on rental.customer_id = c.customer_id
        left join address
        on address.address_id = (
            select address_id from store where store.store_id = c.store_id
        )
        group by c.customer_id, address.address
        ```