jqr
=======



[![Build Status](https://travis-ci.org/ropensci/jqr.png?branch=master)](https://travis-ci.org/ropensci/jqr)
[![Coverage Status](https://coveralls.io/repos/ropensci/jqr/badge.svg?branch=master)](https://coveralls.io/r/ropensci/jqr?branch=master)

R interface to jq http://stedolan.github.io/jq/

To install, after cloning the repo the first time, run (from the command line)

```
./bootstrap.R
```

which will download the 1.4 release of [jq](http://stedolan.github.io/jq/).  This does not need to be run on subsequent runs, and will stop being required once [this issue](https://github.com/ropensci/jqr/issues/1) is resolved.


```r
library("jqr")
```

## Interfaces

### low level

There's a low level interface in which you can execute `jq` code just as you would on the command line:


```r
str <- '[{
    "foo": 1,
    "bar": 2
  },
  {
    "foo": 3,
    "bar": 4
  },
  {
    "foo": 5,
    "bar": 6
}]'
```


```r
jq_(str, ".[]")
#> [1] "{\"foo\":1,\"bar\":2}" "{\"foo\":3,\"bar\":4}" "{\"foo\":5,\"bar\":6}"
```


```r
jq_(str, "[.[] | {name: .foo} | keys]")
#> [1] "[[\"name\"],[\"name\"],[\"name\"]]"
```

### high level 

The other is higher level, and uses a suite of functions to construct queries. Queries are constucted, then excuted with the function `jq()`. 

Examples:

Index


```r
x <- '[{"message": "hello", "name": "jenn"}, {"message": "world", "name": "beth"}]'
x %>% index() %>% jq
#> {"message":"hello","name":"jenn"} {"message":"world","name":"beth"}
```

Sort


```r
'[8,3,null,6]' %>% sort %>% jq
#> [null,3,6,8]
```

reverse order


```r
'[1,2,3,4]' %>%  reverse %>% jq
#> [4,3,2,1]
```

#### string operations

join


```r
'["a","b,c,d","e"]' %>% join %>% jq
#> "a, b,c,d, e"
'["a","b,c,d","e"]' %>% join(`;`) %>% jq
#> "a; b,c,d; e"
```

ltrimstr


```r
'["fo", "foo", "barfoo", "foobar", "afoo"]' %>% index() %>% ltrimstr(foo) %>% jq
#> "fo" "" "barfoo" "bar" "afoo"
```

rtrimstr


```r
'["fo", "foo", "barfoo", "foobar", "foob"]' %>% index() %>% rtrimstr(foo) %>% jq
#> "fo" "" "bar" "foobar" "foob"
```

startswith


```r
'["fo", "foo", "barfoo", "foobar", "barfoob"]' %>% index %>% startswith(foo) %>% jq
#> false true false true false
```

endswith


```r
'["fo", "foo", "barfoo", "foobar", "barfoob"]' %>% index %>% endswith(foo) %>% jq
#> false true true false false
```

tojson, fromjson, tostring


```r
'[1, "foo", ["foo"]]' %>% index %>% tostring %>% jq
#> "1" "foo" "[\"foo\"]"
'[1, "foo", ["foo"]]' %>% index %>% tojson %>% jq
#> "1" "\"foo\"" "[\"foo\"]"
'[1, "foo", ["foo"]]' %>% index %>% tojson %>% fromjson %>% jq
#> 1 "foo" ["foo"]
```

contains


```r
'"foobar"' %>% contains("bar") %>% jq
#> true
```

unique


```r
'[1,2,5,3,5,3,1,3]' %>% unique %>% jq
#> [1,2,3,5]
```

#### types

get type information for each element


```r
'[0, false, [], {}, null, "hello"]' %>% types %>% jq
#> ["number","boolean","array","object","null","string"]
'[0, false, [], {}, null, "hello", true, [1,2,3]]' %>% types %>% jq
#> ["number","boolean","array","object","null","string","boolean","array"]
```

select elements by type


```r
'[0, false, [], {}, null, "hello"]' %>% index() %>% type(booleans) %>% jq
#> false
```

#### key operations

get keys


```r
str <- '{"foo": 5, "bar": 7}'
str %>% keys() %>% jq
#> ["bar","foo"]
```

delete by key name


```r
str %>% del(bar) %>% jq
#> {"foo":5}
```

check for key existence


```r
str3 <- '[[0,1], ["a","b","c"]]'
str3 %>% haskey(2) %>% jq
#> [false,true]
str3 %>% haskey(1,2) %>% jq
#> [true,false,true,true]
```

#### Maths


```r
'{"a": 7}' %>%  do(.a + 1) %>% jq
#> 8
'{"a": [1,2], "b": [3,4]}' %>%  do(.a + .b) %>% jq
#> [1,2,3,4]
'{"a": [1,2], "b": [3,4]}' %>%  do(.a - .b) %>% jq
#> [1,2]
'{"a": 3}' %>%  do(4 - .a) %>% jq
#> 1
'["xml", "yaml", "json"]' %>%  do('. - ["xml", "yaml"]') %>% jq
#> ". - [\"xml\", \"yaml\"]"
'5' %>%  do(10 / . * 3) %>% jq
#> 6
```

comparisons


```r
'[5,4,2,7]' %>% index() %>% do(. < 4) %>% jq
#> false false true false
'[5,4,2,7]' %>% index() %>% do(. > 4) %>% jq
#> true false false true
'[5,4,2,7]' %>% index() %>% do(. <= 4) %>% jq
#> false true true false
'[5,4,2,7]' %>% index() %>% do(. >= 4) %>% jq
#> true true false true
'[5,4,2,7]' %>% index() %>% do(. == 4) %>% jq
#> false true false false
'[5,4,2,7]' %>% index() %>% do(. != 4) %>% jq
#> true false true true
```

length


```r
'[[1,2], "string", {"a":2}, null]' %>% index %>% length %>% jq
#> 2 6 1 0
```

sqrt


```r
'9' %>% sqrt %>% jq
#> 3
```

floor


```r
'3.14159' %>% floor %>% jq
#> 3
```

find minimum


```r
'[5,4,2,7]' %>% min %>% jq
#> 2
'[{"foo":1, "bar":14}, {"foo":2, "bar":3}]' %>% min %>% jq
#> {"foo":2,"bar":3}
'[{"foo":1, "bar":14}, {"foo":2, "bar":3}]' %>% min(foo) %>% jq
#> {"foo":1,"bar":14}
'[{"foo":1, "bar":14}, {"foo":2, "bar":3}]' %>% min(bar) %>% jq
#> {"foo":2,"bar":3}
```

find maximum


```r
'[5,4,2,7]' %>% max %>% jq
#> 7
'[{"foo":1, "bar":14}, {"foo":2, "bar":3}]' %>% max %>% jq
#> {"foo":1,"bar":14}
'[{"foo":1, "bar":14}, {"foo":2, "bar":3}]' %>% max(foo) %>% jq
#> {"foo":2,"bar":3}
'[{"foo":1, "bar":14}, {"foo":2, "bar":3}]' %>% max(bar) %>% jq
#> {"foo":1,"bar":14}
```

## Meta

* Please [report any issues or bugs](https://github.com/ropensci/jqr/issues).
* License: MIT
* Get citation information for `jqr` in R doing `citation(package = 'jqr')`

[![rofooter](http://ropensci.org/public_images/github_footer.png)](http://ropensci.org)
