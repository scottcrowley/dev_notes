# MySQL Database Design

### Notes on [MySQL Database Design](https://laracasts.com/series/mysql-database-design) from Laracasts and other Miscellaneous MySQL notes

* ## Sakila Example database
    * Throughout the serier, JW uses the [Sakila example database](https://dev.mysql.com/doc/sakila/en/) as a reference. Download the zip file from https://downloads.mysql.com/docs/sakila-db.zip
* ## Foreign Keys
    * A foreign key constraint can be added after a table has already been created by using
        ```sql
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
        ```sql
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
        ```sql
        count(rentals.rental_id) rentals_checked_out
        ```
    * Example from series using Sakila DB
        ```sql
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
        ```sql
        group by customer.customer_id, rentals.rental_id
        ```
* ## Sub Queries and Multiple Joins
    * Anytime you reference a table in a from clause, you can substitute the table with a sub query. Important to note that the sub query requires an alias.
        ```sql
        select round(avg(total_rentals)) as average_rentals
        from (
            selecet date(rental_date) as day, count(*) as total_rentals
            from rental
            group by day
            order by day desc
        ) rentals
        ```
    * Sub queries are more time intensive then performing multiple joins. Might be better to use multiple joins in some cases instead of a sub query
    * You can use a sub query in place of any field or table reference. The following sql takes about 110 ms to run.
        ```sql
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
        ```sql
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
    * Another example of using a sub query: You need to find the top 10 users that have the most read posts. There is a users table, post table and a post_reads table that is a pivot table for when a user reads a post. The post_reads table has a user_id and post_id column.
        ```sql
        select users.id, users.name, count(*) as posts_read
        from users
        left join post_reads
        on post_reads.post_id in (
            select id from posts where user_id = users.id
        )
        group by users.id
        order by posts_read desc
        limit 10
        ```
        The sub query is the section of the statement that begins with `in`. So the query states: first select all `posts` where the `user_id` of the `post` `=` the currently found `user` from the original `select` statement. Then `left join` the the `post_reads` table on the condition that the `post_id` `=` all the results from the sub query. Then `count` the results of that join and assign the value to `posts_read`.
    * **Using Sub Queries with Laravel 6**
        * You can now do sub queries on a model or the `DB` class by using the `addSelect()` method or by passing a closure to the `orderBy()` and `from()` methods of the query builder. 
            * Examples from Laracast lesson [What's New in Laravel 6: Eloquent Subquery Additions](https://laracasts.com/series/whats-new-in-laravel-6/episodes/4). He is using a `users` table and a `watching` table, which contains the `user_id` along with the `lesson_id` (video), `created_at` & `updated_at`.

                * We want to find the most recently watched lesson for user_id=41 & user_id=50. 
                    ```php
                    App\User::addSelect(
                        [
                            'last_watched_video' => function($query) {
                                $query->select('lesson_id')
                                    ->from('watching')
                                    ->whereColumn('user_id', 'users.id')
                                    ->limit(1)
                                    ->latest();
                            }
                        ]
                    )->find([41, 50]);
                    ```
                    `last_watched_video` is the alias to be used in the results. You need to use the `whereColumn` method when linking the watching table back to the `User` model. This is saying `where watching.user_id = users.id`. Note the second argument references the `users` table and not the `User` model.
                    * The actual sql query being executed above looks like this:
                        ```sql
                        select `users`.*, 
                            (select `lesson_id` 
                                from `watching` 
                                where `user_id` = `users`.`id` 
                                order by `created_at` desc 
                                limit 1
                            ) as `last_watched_video` 
                        from `users` 
                        where `users`.`id` in (?,?)
                        ```
                    * Note: In place of the closure function above, you could use a model, if one is available, which it was not in this case.
                        ```php
                        App\User::addSelect(
                            [
                                'last_watched_video' => App\Watching::select(
                                    //whatever the select statement should be
                                )
                            ]
                        );
                        ```
                * Next example we want to grab his users but order them by most recently watched videos being displayed first. So, someone that hasn't watched a video in a very long time, will be at the bottom of the results and someone that just watched a video, will be at the top.
                    ```php
                    App\User::orderByDesc(function($query) {
                        $query->select('created_at')
                            ->from('watching')
                            ->latest()
                            ->whereColumn('user_id', 'users.id')
                            ->limit(1);
                    })->find([41, 50]);
                    ```
                    * The actual sql query being executed above looks like this:
                        ```sql
                        select *
                        from `users`
                        where `users`.`id` in (?,?)
                        order by (
                            select `created_at` 
                                from `watching` 
                                where `user_id` = `users`.`id` 
                                order by `created_at` desc 
                                limit 1
                        ) desc
                        ```
* ## Recursive Query
    * A recursive query is used when you have a foreign key that references its own table. This can give you the parent to child hierarchy. Using the Lego project as an example, the `themes` table contains the following columns: `id`, `name`, `parent_id`. `parent_id` has a constraint on the `id` column. This allows for a theme to have many child or parent themes associated with it. If the `parent_id` column is null then that is the top parent theme. 
    * Below is a query I found on http://www.mysqltutorial.org/mysql-adjacency-list-tree/. I modified it to fit my needs. Not sure if it is too resource intensive to be useful.
        
        The `where` clause at the end contains the theme we want to query. In this case it is 175. It also concatenates a label that shows the hierarchy of all related themes, where 175 is at the bottom of it.
        ```sql
        WITH RECURSIVE theme_hierarchy (id, name, parent, lvl, parent_label) AS
            (
                SELECT id, name, parent_id parent, 1 lvl, name as parent_label
                FROM themes
                WHERE parent_id is NULL
                    UNION ALL
                    SELECT t.id, t.name, t.parent_id, lvl+1, CONCAT(th.parent_label, ' -> ', t.name)
                    FROM themes t
                    INNER JOIN theme_hierarchy th ON th.id = t.parent_id
            )
        SELECT parent_label
        FROM theme_hierarchy th
        WHERE th.id = 175
        ORDER BY lvl DESC
        ```

        I executed this code in Laravel using the DB::raw facade. It was being used within a chunk function when all sets were being retrieved.
        ```php
        DB::table('sets')
            ->select(
                'sets.*',
                'set_image_urls.image_url',
                'themes.name as theme_name',
                'themes.parent_id as theme_parent_id'
            )
            // ->where('year', 2019)
            ->orderBy('set_num')
            ->leftJoin('themes', 'sets.theme_id', '=', 'themes.id')
            ->leftJoin('set_image_urls', 'sets.set_num', '=', 'set_image_urls.set_num')
            ->chunk(1000, function ($chunk) use (&$sets) {
                // $chunk = $this->getThemeHeirarchy($chunk);
                $chunk->map(function ($item, $key) {
                    $setTheme = DB::select(
                        DB::raw('WITH RECURSIVE theme_hierarchy (id, name, parent, lvl, parents_label) AS 
                            (
                                SELECT id, name, parent_id parent, 1 lvl, name as parents_label 
                                FROM themes 
                                WHERE parent_id is NULL 
                                UNION ALL 
                                SELECT t.id, t.name, t.parent_id, lvl+1, CONCAT(th.parents_label, \' -> \', t.name) 
                                FROM themes t 
                                INNER JOIN theme_hierarchy th ON th.id = t.parent_id
                            ) 
                            SELECT id, name, parent, lvl, parents_label 
                            FROM theme_hierarchy th 
                            WHERE th.id = :theme_id 
                            ORDER BY lvl DESC'),
                        ['theme_id' => $item->theme_id]
                    );
                    $item->theme_label = $setTheme[0]->parents_label;
                });
                $sets = $sets->merge($chunk);
            });
        ```
* ## Miscellaneous
    * Column aliases can be assigned with or without the `as`
        ```sql
        address.address store_address
        address.address as store_address
        ```
    * To alias a table, make sure to update all occurenences of the full table name or an error will be thrown. Example: Change the `customer` table name to `c`.
        ```sql
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
    * The `HAVING` clause is the same as the `WHERE` clause but can be used on aggregated columns. In the example below, using the Sakila DB, they wanted to write a query to select all title that have $200 in sales or more. Since aliases are used on the aggregated colums, then the `WHERE` clause will not work. You must use the `HAVING` clause. Also, `count(*)` can be used since it counts all the rows from the `group by` clause. So if you need to filter rows down based on an aggregated column then you need to use the `HAVING` clause instead of the `WHERE` clause.
        ```sql
        select title, sum(amount) sales, count(*) rentals, from rental
        join payment
        on payment.rental_id = rental.rental_id
        join inventory
        on inventory.inventory_id = rental.inventory_id
        join film
        on film.film_id = inventory.film_id
        group by title
        having sales > 200
        order by sales desc
        ```
