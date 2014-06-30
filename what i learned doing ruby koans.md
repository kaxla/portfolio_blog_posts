#Default Hash Values in Ruby

###Hashes can have default values. Whatever you put in that default will automatically be the value for any nonexistant key.

```ruby
hash = Hash.new("puppies")
```

means that 

```ruby
hash[:nonexistantkey] = "puppies"
```

Ok, that's pretty straightforward. But what happens when you try to do something more complicated? 

**What if you set your default value to an empty array?**

```ruby
hash = Hash.new([])`` As expected ``hash[:notakey] = []
```

**What if you shovel some values into that empty array?**

```ruby
hash[:one] << "uno"`` As expected ``hash[:one] = ["uno"]
```

**What if you shovel some more values into that array?**

```ruby
hash[:two] << ["dos"]
```

Here's where it gets weird. You would now expect 

```ruby
hash[:two] = ["dos"]
``` 
and 
```ruby
hash[:one] = ["uno"]
```

but they don't:

```ruby
hash[:one] = ["uno", "dos"] 

hash[:two] = ["uno", "dos"]

hash[:notakey] = ["uno", "dos"]
```

PLUS even after all of this 
```ruby
hash = {}
```

##WAT?

That default empty array value is not setting each key to it's own array, it is setting all keys to THE SAME array.  Hash is still equal to an empty hash because you never actually assigned a value to the keys ``:one`` and ``:two``, you just shovelled everything into the default array which only returns if you input a nonexistant key.

**Okay, so what if instead of using an empty array as the default you use a block that assigns an empty array?**

```ruby
hash = Hash.new {|hash, key| hash[key] = []}
```

now if you call an unknown key it will evaluate the block and return the answer. So 
```ruby
hash[:notakey] = []
```

You might expect it to have the same behavior as above when shovelling values into the array, but it doesn't. It has the behavior we expected the above to have: 

```ruby
hash[:one] << "uno"

hash[:two] << "dos"
```

now

```ruby
hash[:one] = ["uno"]

hash[:two] = ["dos"]

hash[:notakey] = []

hash = {:one => ["uno"], :two => ["dos"]}
```

The block is assigning each key its own empty array and actually assigns the value instead of just shovelling it into the default array.

There's a blog post about all kinds of cool stuff you can do with these techniques [here](http://endofline.wordpress.com/2010/12/24/hash-tricks/)

Original problem from [Ruby Koans](http://rubykoans.com/)

Original help figuring out what the heck was happening with the default value of empty array thanks to [this](http://stackoverflow.com/questions/2698460/strange-ruby-behavior-when-using-hash-new) stack overflow post
