## Authentication

Most requests require the credentials of a Cloudant account. There are
two ways to provide those credentials. They can either be provided using
HTTP Basic Auth or as an HTTP cookie named AuthSession.

### Basic Authentication

```shell
curl -u $USERNAME https://$USERNAME.cloudant.com
```

```python
import cloudant

url = "https://{0}:{1}@{0}.cloudant.com".format(USERNAME, PASSWORD)
account = cloudant.Account(url)
ping = account.get()
print ping.status_code
# 200
```

```javascript
var nano = require('nano');
var account = nano("https://$USERNAME:$PASSWORD@$USERNAME.cloudant.com");

account.request(function (err, body) {
  if (!err) {
    console.log(body);
  }
});
```

With Basic authentication, you pass along your credentials as part of every request.

### Cookies

The cookie can be obtained by performing a POST request to `/_session`. With the cookie
set, information about the logged in user can be retrieved with a GET
request. With a DELETE request you can end the session. Further
details are provided below.

  -------------------------------------------------------------------------
  Meth Path   Description             Headers                     Form
  od                                                              Parameter
                                                                  s
  ---- ------ ----------------------- --------------------------- ---------
  GET  /\_ses Returns cookie based    AuthSession cookie returned ---
       sion   login user information  by POST request             

  POST /\_ses Do cookie based user    `Content-Type: application/ name,
       sion   login                   x-www-form-urlencoded`      password

  DELE /\_ses Logout cookie based     AuthSession cookie returned ---
  TE   sion   user                    by POST request             
  -------------------------------------------------------------------------

Here is an example of a post request to obtain the authentication
cookie.

```http
name=YourUserName&password=YourPassword
```

And this is the corresponding reply with the Set-Cookie header.

```http

```

Once you have obtained the cookie, you can make a GET request to obtain
the username and its roles:

The body of the reply looks like this:

```http

```

To log out, you have to send a DELETE request to the same URL and sumbit
the Cookie in the request.

```http

```

This will result in the following response.

```http

```


```shell
# get cookie
curl https://$USERNAME.cloudant.com/_session \
     -X POST \
     -c /path/to/cookiefile
     -d "name=$USERNAME&password=$PASSWORD"

# use cookie
curl https://$USERNAME.cloudant.com/_session \
     -b /path/to/cookiefile
```

```python
import cloudant

account = cloudant.Account(USERNAME)
login = account.login(USERNAME, PASSWORD)
print login.status_code
# 200
logout = account.logout()
print logout.status_code
# 200
all_dbs = account.all_dbs()
print all_dbs.status_code
# 401
```

```javascript
var nano = require('nano');
var cloudant = nano("https://"+$USERNAME+".cloudant.com");
var cookies = {}

cloudant.auth($USERNAME, $PASSWORD, function (err, body, headers) {
  if (!err) {
    cookies[$USERNAME] = headers['set-cookie'];
    cloudant = nano({
      url: "https://"+$USERNAME+".cloudant.com",
      cookie: cookies[$USERNAME] 
    });

    // ping to ensure we're logged in
    cloudant.request({
      path: 'test_porter'
    }, function (err, body, headers) {
      if (!err) {
        console.log(body, headers);
      }
    }); 
  }
});
```

With Cookie authentication, you use your credentials to acquire a cookie which remains active for twenty four hours. You send the cookie with all requests until it expires.

Logging out causes the cookie to expire immediately.
