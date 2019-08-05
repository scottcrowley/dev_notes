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