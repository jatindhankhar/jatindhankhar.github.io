---
title: 'Auto Refresh tokens in Ruby using Procs  '
categories: blog
excerpt: Auto refresh API tokens seamlessly in ruby using procs.
tags:
- ruby
- programming
comments: true
description: Auto refresh API tokens seamlessly in ruby using procs.

---
# Backstory ?

Few months ago I had to integrate a third party service into our system.  
The third party in question was used in a dashboard to verify certain details of users.

So in an ideal world, the end user just clicks on verify and our system handles rest of the stuff (like it should). 

So the service had an api which can be used to verify those details (Obviously :smile:) but it used a refresh token which expired every 15 minutes.

[Refresh tokens are good from security point of view](https://auth0.com/learn/refresh-tokens/) but using them in normal flow can be tricky sometimes since they are short lived and cannot be reused.

# How ?

We needed to handle the refresh of the token gracefully without impacting the original request.

The verification call was wrapped in a proc, so that it be can be passed around.

If  the request returned 401, unauthorised, we could regenerate the token and retry the original request.

```ruby

class APIClient

   class RefreshTokenError < Exception
     def initialize(msg = "Failed to Refresh Token")
       super
     end
   end

   AUTH_ERROR_CODE = 400
   SERVER_ERROR_CODE = 520
   ERROR_STATUS = "ERROR"
   BASE_URL = "https://api-domain.tld"
   RETRY_THRESHOLD = 3 
   CLIENT_ID = ENV['CLIENT-ID']
   CLIENT_SECRET = ENV['CLIENT-SECRET']

   def initialize
     @token = nil
     @retries = 0
   end

   def refresh_and_execute(request)
     response = request.call
     return response if auth_success?(response)
     while @retries < RETRY_THRESHOLD
       return request.call if refresh_token
       @retries += 1
     end
     raise RefreshTokenError
   end

   def validate_details(query)
     request_call = Proc.new {HTTParty.get(validation_endpoint, query: query, headers: token_headers, timeout: 12)}
     refresh_and_execute(request_call)
   end

   def auth
     response = HTTParty.post(authorization_endpoint, headers: auth_headers)
     token_updated = false
     if response.success?
       unless response.parsed_response.nil?
         @token = response.parsed_response.fetch("data", {}).fetch("token", nil)
         token_updated = !@token.nil?
       end
       @retries = 0 if token_updated
     end
     token_updated
   end

   private

   def refresh_token
     auth
   end

   def authorization_endpoint
     URI.join(BASE_URL, "authorize_endpoint").to_s
   end

   def validation_endpoint
     URI.join(BASE_URL, "verificationUrl")
   end

   def auth_headers
     {
         'X-Client-Id' => CLIENT_ID,
         'X-Client-Secret' => CLIENT_SECRET
     }
   end

   def token_headers
     {'Authorization' => "Bearer #{@token}"}
   end

   def auth_success? response
     response.success? and response.parsed_response["status"] != ERROR_STATUS and not (response.parsed_response["subCode"].to_i.between?(AUTH_ERROR_CODE, SERVER_ERROR_CODE))
   end
 end
```

The main magic (or just plain logic :sweat_smile) happens in this

```ruby

def refresh_and_execute(request)
      response = request.call
      return response if auth_success?(response)
      while @retries < RETRY_THRESHOLD
        return request.call if refresh_token
        @retries += 1
      end
      raise RefreshTokenError
end
```

It returns the original response if there was no authentication error.

Otherwise it attempts to refresh the token and retry the original request, until retry threshold is reached.