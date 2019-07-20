---
title: Custom Http Header and Ruby Standard Library
categories: programming
excerpt: 'Adventures with using custom http header and how ruby''s NET::HTTP library
  parses it '
tags:
- ruby
- programming
comments: true
description: 'Adventures with using custom http header and how ruby''s NET::HTTP library
  parses it '

---
# The problem

One day at work I got an escalation from one of the third-party vendors that all the api calls to them were silently getting rejected on their end. They provided the explanation that one of the HTTP header that was used to supplying the api key itself, (let's call it  `API-KEY`)  was being sent incorrectly.

They wanted the header to be a lower case like `api-key` but we started standing it in upper case `API-KEY`. This should not happen since we had a  patch in place, to handle this situation.


## Little background about HTTP headers and NET::HTTP

As per the RFC, http headers are case-insensitive, which means the destination application should be able to understand both the uppercase and lowercase.

Since for most of the applications, http header is case-insensitive. Ruby's standard networking library NET::HTTP converts every header to uppercase which is not an issue for most of the third party apis.

Many popular libraries like Httparty use Net::HTTP as backend.

But sometimes we need to send a particular http header as is, without any modifications.

To prevent http header keys from being modified by NET::HTTP, I used this solution from [https://calvin.my/posts/force-http-header-name-lowercase](https://calvin.my/posts/force-http-header-name-lowercase "https://calvin.my/posts/force-http-header-name-lowercase")

which worked fine.

```ruby
class ImmutableKey < String 
         def capitalize 
               self 
         end 
 end
```

and using the above key as  follows

```ruby
{ ImmutableKey.new("api-key"): 'SECRET-KEY' } 
```

As per the third party, this started occurring from a particular time and then it occurred to me, the timeline matches perfectly with our rails upgrade from rails 4 to rails 5. 
To verify the hunch, I ran the same code again in both rails 4 and rails 5 boxes with HTTParty debug log on and yep, there it was. Headers were being capitalized in new rails boxes.

```bash
# Rails 4 box
opening connection to thirdparty.com:443...
starting SSL for thirdparty.com:443...
SSL established

<- POST "/endpoint HTTP/1.1\r\napi-key: 'SECRET'
```

```bash
# Rails 4 box
opening connection to thirdparty.com:443...
starting SSL for thirdparty.com:443...
SSL established

<- POST "/endpoint HTTP/1.1\r\nAPI-KEY: 'SECRET'
```

But why did it happen, my first thought was to check for gem version of httparty and even though there was a bump from `.0.14` to `0.17`, further debugging proved that httparty was not the issue and mutation of forms were happening at `Net::HTTP` level.

[https://github.com/jnunemaker/httparty/blob/99751ac98af929b315c74c2ac0f5ffa09195f7ae/lib/httparty/request.rb#L213](https://github.com/jnunemaker/httparty/blob/99751ac98af929b315c74c2ac0f5ffa09195f7ae/lib/httparty/request.rb#L213 "https://github.com/jnunemaker/httparty/blob/99751ac98af929b315c74c2ac0f5ffa09195f7ae/lib/httparty/request.rb#L213")

```ruby
def setup_raw_request
          @raw_request = http_method.new(request_uri(uri))
          @raw_request.body_stream = options[:body_stream] if options[:body_stream]
    
          if options[:headers].respond_to?(:to_hash)
            headers_hash = options[:headers].to_hash
    
            @raw_request.initialize_http_header(headers_hash)
            # If the caller specified a header of 'Accept-Encoding', assume they want to
            # deal with encoding of content. Disable the internal logic in Net:HTTP
            # that handles encoding, if the platform supports it.
            if @raw_request.respond_to?(:decode_content) && (headers_hash.key?('Accept-Encoding') || headers_hash.key?('accept-encoding'))
              # Using the '[]=' sets decode_content to false
              @raw_request['accept-encoding'] = @raw_request['accept-encoding']
            end
          end
```

specifically this line

`@raw_request.initialize_http_header(headers_hash`

So if `NET::HTTP` why did rails upgrade break it ?

It was ruby version upgrade, earlier we were using ruby `2.1.3` and with the rails we jumped to ruby `2.5.2` which means standard library also had some changes.

So let see the diff between of

Older ruby version `2.1.3` had

[https://github.com/ruby/ruby/blob/6d728bdae9de565ad9d0b2fee2d4c2a33c6f4eac/lib/net/http/header.rb#L162]()

```ruby
def each_capitalized
    block_given? or return enum_for(__method__)
    @header.each do |k,v|
      yield capitalize(k), v.join(', ')
    end
  end

  alias canonical_each each_capitalized

  def capitalize(name)
    name.split(/-/).map {|s| s.capitalize }.join('-')
  end
```

while ~~ruby~~ `~~2.5~~` ~~introduced~~ this commit [https://github.com/ruby/ruby/commit/1a98f56ae14724611fc8f7c220e470d27f6b57e4](https://github.com/ruby/ruby/commit/1a98f56ae14724611fc8f7c220e470d27f6b57e4 "https://github.com/ruby/ruby/commit/1a98f56ae14724611fc8f7c220e470d27f6b57e4") introduced some changes  to underlying `captialize` method by using `to_s` 

```ruby
    name.to_s.split(/-/).map {|s| s.capitalize }.join('-')
```

which caused our ImmutableString class to return a new string object instead an object of ImmutableString class with capitalized frozen

```ruby
ImmutableKey("new").class # ImmutableKey
ImmutableKey("new").to_s.class # String
ImmutableKey("new").to_str.class # String

```

# Fix 

Fix was to make the `to_s` and `to_str` return the `self` so that the returned object is an instance of `ImmutableKey` instead of the base string class

```ruby
class ImmutableKey < String 
         def capitalize 
               self 
         end 
         
         def to_s
          self 
         end 
         
         alias_method :to_str, :to_s
 end
```


Debugging the issue was fun though :D
