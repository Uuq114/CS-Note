# Ruby-intro

[toc]

## built-in function

**String**

substring

```ruby
my_word = "helloruby"
puts my_word[2,4] # idx: 2, len: 4
# output: llor
```

indexAt

```ruby
my_word = "helloruby"
puts my_word.index("llo") # 2
```

substring existence

```ruby
my_phrase = "helloworld"
puts my_phrase.include? "world" # true
```

toString

```ruby
num = 20
puts ("my num is:" + num.to_s)
```



**Array**

init

```ruby
friends = Array["kevin", "tom", "chen"]
```



**Hash**

init

```ruby
states = {
    "hubei" => "hb",
    "shanghai" => "sh",
    "beijing" => "bj",
}
puts states["hubei"]
```



**function**

define

```ruby
def some_func([some_arg=default_value, ...])
    # do sth
end
```



**if-else**

if...elsif...else...end