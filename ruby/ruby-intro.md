# Ruby-intro

[toc]

## base conceptss

### String

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



### Array

init

```ruby
friends = Array["kevin", "tom", "chen"]
```



### Hash

init

```ruby
states = {
    "hubei" => "hb",
    "shanghai" => "sh",
    "beijing" => "bj",
}
puts states["hubei"]
```



### function

define

```ruby
def some_func([some_arg=default_value, ...])
    # do sth
end
```



### procedure control

if-else

```
if...elsif...else...end
```

"switch"

```
case...when...end...when...end
```

for-loop

```ruby
# 1
for...in...end
# 2
arr = [...]
arr.each do |elem|
    # do sth
end
# 3
for idx in 0..10
    # do sth
end
# 4
6.times do |idx|
    # do sth
end
```



### read/write file

read

```ruby
# 1
File.open("list.txt", "r") do |file|
    # do sth
end
# 2
file = File.open("list.txt", "r")
```



### exception

"try-catch"

```ruby
begin
    # some error
rescue
    # handle error
end
```



### class

define class

```ruby
class Book
    attr_accessor :title, :author, :pages
    def initialize(title, author, pages)
    	@title = title
        # ...
    end
end

book1 = Book.new("title1", "author1", "pages1")
```

inherience

child class can overwrite father class' function

```ruby
class Animal
    def move
        # ...
    end
end

class Dog < Animal
    def run
        # ...
    end
end
```



### module

module rb file

```ruby
module
	# ...
end
```

caller rb file

```ruby
require_relative "xxx.rb"
include ...

# call func
```

