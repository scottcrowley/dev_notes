# Redis

## Notes from the "Let’s Build a Forum with Laravel and TDD" starting in lesson 66

* Install and setup
    * Laravel requires predis. Install using
        * `composer require predis/predis`
    * Install Redis using home-brew
        * `brew install redis`
    * Once it is installed, make sure its running by using:
        * `redis-server` This starts the redis server.
        * or `brew services start redis`
    * Enter command line mode:
        * `redis-cli`
    * Laravel uses the facade `Redis::`, which needs to be imported in each doc where it’s being used.
        * `use Illuminate\Support\Facades\Redis;`
* Commands:
    * `zincrby(key, increment, member)` - increments the store for a certain identity
    * `zrevrange(key, start, stop)` - returns a range of results from the store in rev order. If you want all results then use -1 for the stop param
