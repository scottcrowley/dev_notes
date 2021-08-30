# Miscellaneous Testing

## PHP Testing Jargon
##### The following notes are from the Laracast episode [When to Reach for Data Providers](https://laracasts.com/series/php-testing-jargon/episodes/4) in the series [PHP Testing Jargon](https://laracasts.com/series/php-testing-jargon)

### Using the setUp method
When using the setUp method with in the test class you can removed duplicated code and assign something to a property on the class.
```php
protected TagParser $parser;

protected function setUp(): void
{
    $this->parser = new TagParser();
}
```
***IMPORTANT:*** You have to be explicit with the "Return Type" being used. This is done by using `: void` after the method name.

### Data Providers
Data Providers can be used to simplify code where the same assertion is being made but there are different data sets to test against.

In the episode the same assertion is being made in several test that performs the same function call but are using different data sets and expected results. 
```php
public function test_it_parses_a_single_tag()
{
    $result = $this->parser->parse("personal");
    $expected = ["personal"];

    $this->assertSame($expected, $result);
}

public function test_it_parses_a_comma_separated_list_of_tags()
{
    $result = $this->parser->parse("personal, money, family");
    $expected = ["personal", "money", "family"];

    $this->assertSame($expected, $result);
}
```
The function call `$result = $this->parser->parse(` is being used in the each test as well as the same assertion `$this->assertSame($expected, $result);`

This can be simplified by using one test that utilizes Data Providers.
```php
/**
 * @dataProvider tagsProvider
 */
public function test_it_parses_tags($input, $expected)
{
    $parser = new TagParser();

    $result = $parser->parse($input);

    $this->assertSame($expected, $result);
}
```
The new test `test_it_parses_tags` replaces all other tests that are using the previous duplicated assertions. ie. `test_it_parses_a_single_tag` and `test_it_parses_a_comma_separated_list_of_tags`
The actual Data Provider is being referenced in the comment tags of the test itself. `@dataProvider tagsProvider` is used to tell the test to use a function called `tagesProvider` to get the input data as well as the expected result when run against the `$parser->parse` function.
```php
public function tagsProvider()
{
    return [
        ["personal", ["personal"]],
        ["personal, money, family", ["personal", "money", "family"]],
        ["personal,money,family", ["personal", "money", "family"]],
        ["personal | money | family", ["personal", "money", "family"]],
        ["personal|money|family", ["personal", "money", "family"]],
        ["personal!money!family", ["personal", "money", "family"]],
    ];
}
```
Each element of the root array is a different assertion to be made. In the assertion array is the value to be used for the `$input` value and the second array is the expected result (`$expected`) to test against.

The only potential downside to using Data Providers is that you lose the description of what each dataset is testing for.

