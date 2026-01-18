---
title: "Blocking WordPress Brute-Force Attacks With Nginx"
modified_date: "2026-01-18"
last_modified_at: "2026-01-18"
redirect_from: "/2016/07/server-logs-explained-part-3"
---
There are a few different ways that crackers will try to get into your WordPress installation, and one of them is by using a plain old brute-force attack. This kind of attack requires nothing more than a freely available exploit toolkit, and is not difficult to detect in the server logs. In the first section of this post, I'm going to give an example of what a brute force attack looks like, and then to make things more interesting I'll discuss some techniques used to mitigate them using Nginx.

## Example Server Logs

As you would guess, when one computer makes hundreds of requests for a resource in quick succession, it leaves some pretty serious traces in the server logs (these are real logs, but I removed the server name):
```
203.0.113.42 - - [22/Jun/2016:19:18:58 -0700] "POST /wp-login.php HTTP/1.1" 200 3848 "https://www.example.com/wp-login.php" "Mozilla/4.0 (compatible; MSIE 9.0; Windows NT 6.1; 125LA; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)"
203.0.113.42 - - [22/Jun/2016:19:18:59 -0700] "POST /wp-login.php HTTP/1.1" 200 3848 "https://www.example.com/wp-login.php" "Mozilla/4.0 (compatible; MSIE 9.0; Windows NT 6.1; 125LA; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)"
203.0.113.42 - - [22/Jun/2016:19:18:59 -0700] "POST /wp-login.php HTTP/1.1" 429 0 "https://www.example.com/wp-login.php" "Mozilla/4.0 (compatible; MSIE 9.0; Windows NT 6.1; 125LA; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)"
203.0.113.42 - - [22/Jun/2016:19:19:00 -0700] "POST /wp-login.php HTTP/1.1" 429 0 "https://www.example.com/wp-login.php" "Mozilla/4.0 (compatible; MSIE 9.0; Windows NT 6.1; 125LA; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)"
203.0.113.42 - - [22/Jun/2016:19:19:00 -0700] "POST /wp-login.php HTTP/1.1" 429 0 "https://www.example.com/wp-login.php" "Mozilla/4.0 (compatible; MSIE 9.0; Windows NT 6.1; 125LA; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)"
203.0.113.42 - - [22/Jun/2016:19:19:00 -0700] "POST /wp-login.php HTTP/1.1" 429 0 "https://www.example.com/wp-login.php" "Mozilla/4.0 (compatible; MSIE 9.0; Windows NT 6.1; 125LA; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)"
```

Here's a few reasons these requests make it obvious that this is a brute-force attack:
1. They go on for about half an hour. The snippet above is only a small portion of the total logs from this incident.
2. The HTTP method is `POST`, which indicates data is being sent to the server (i.e. the actual password guesses).
3. The resource requested is `/wp-login.php`, which is the default WordPress login page and should rarely be requested, even by legitimate users.

If you look more closely, however, you'll see something interesting: the HTTP response code that the server returns starts off as `200 OK`, but quickly transitions to `429 Too Many Requests`. This is one method of fending off brute force attacks with Nginx.

## Mitigating WordPress Brute-Force Attacks

Fortunately, WordPress brute-force attacks are not that difficult to defend against without the use of plugins or additional software. We can:
1. Restrict access to the login page to a curated list of IP addresses,
2. Explicitly block the IP addresses of known brute-force offenders with Nginx or with a firewall,
3. Password-protect the login page using HTTP Basic Authentication,
4. Or, my personal favorite: Set up rate-limiting with Nginx to cut down on how many requests attackers can make in a certain period of time.

### Restricting Access to Certain IP Addresses

Arguably, the best way to mitigate brute-force attacks is to restrict access to the WordPress login page to only known good IP addresses. Here's what that looks like with Nginx:
```nginx
location = /wp-login.php {
    allow 192.168.1.2;
    allow 192.168.1.50;
    deny all;
    # add your PHP fastcgi config here
}
```

This location block explicitly targets the `/wp-login.php` page and only allows clients using the IP addresses `192.168.1.2` and `192.168.1.50` to access it. All other requests will be met with a `403 Forbidden` error message. Keep in mind you will still need to add your PHP fastcgi config to this location block as well so that Nginx knows to pass legitimate requests back to PHP.

This method ensures that attackers will never get access to the login page, but is difficult to maintain if legitimate WordPress users do not have static IP addresses.

### Denying Access from Certain IP Addresses

Another solution is to explicitly block brute-force offenders. You can block certain IP addresses from accessing the login page with:
```nginx
location = /wp-login.php {
    deny 203.0.113.42;
    # add your PHP fastcgi config here
}
```

If you are familiar with configuring firewalls, you can use firewall commands to block the IP address from accessing anything on your server at all.

While blocking specific IP addresses can be useful, I don't recommend using this as your only line of defense. For one, any IP address used in a brute force attack is almost certainly a VPN, proxy, or bot IP address. By blocking these, you risk denying access to legitimate users, even if that risk is slight. The main concern is that maintaining a list of IP addresses is tedious and unwieldy and is not a good long-term solution. That's not to say this approach is useless, however, as you may want to use it in tandem with another one.

With that in mind, the next possible solution is adding a second layer of protection to the WordPress login page with HTTP Basic Auth.

### Restricting Access Using HTTP Basic Auth

There are two steps to using HTTP Basic Auth with WordPress and Nginx:
1. Create the password file.
2. Configure Nginx.

I am going to skip the first step in this post, as there are many good existing guides on using `openssl` or `apache2-utils` to create a password file.

The second step, configuring Nginx, is fairly simple. Just add two lines to your wp-login location block:
```nginx
location = /wp-login.php {
    auth_basic "Restricted Content";
    auth_basic_user_file /path/to/.password_file;
    # add your PHP fastcgi config here
}
```

You can change "Restricted Content" to any phrase you want, as it will be the message that end-users see when they attempt to access the login page. Make sure you enter the correct path to your password file you created as well.

While password-protecting the login page is a valid solution, it does have the potential to overly complicate the login process for legitimate users.

### Using Rate Limiting in Nginx

Nginx has some [great documentation on how to implement rate limiting](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html), but I am going to provide an example of how to optimize it for WordPress. Setting up rate limiting in Nginx is simple, and only requires two components:
1. We must define a zone in the main `nginx.conf` file.
2. We must implement that zone in the WordPress login location block.

To define the zone, we use `limit_req_zone` and, optionally, `limit_req_status`. These directives go inside the `http` block of the main `nginx.conf` configuration file:
```nginx
http {
     limit_req_zone $binary_remote_addr zone=wordpress:10m rate=15r/m;
     limit_req_status 429;
}
```

The above snippet defines a 10 MB zone named "wordpress" that allows a maximum of 15 requests per minute from any one IP address. The `limit_req_zone` requires a variable, or key. In this case, the key is `$binary_remote_addr`, or the IP address of the client. Nginx will use a maximum of 10 MB of memory to store the keys, and if a key exceeds the maximum number of allowed requests, Nginx will terminate the connection and return the status code defined in `limit_req_status`. The default code is `503 Service Unavailable`, but I prefer the more specific `429 Too Many Requests` response. Keep in mind that Nginx will display a blank page to the client for non-standard HTTP codes if you have not set a custom error page using the `error_page` directive.

You can name the zone anything you want (it is named "wordpress" in the example above) and you can also define any rate limit you feel is appropriate. I found that allowing a maximum of 15 requests per minute is restrictive enough to hamper a brute-force attack but is permissive enough not to interfere with end-users who legitimately mistyped their passwords.

To actually use the zone, we must implement it by adding this code to the WordPress login location block:
```nginx
location = /wp-login.php {
    limit_req zone=wordpress;
    # add your PHP fastcgi config here
}
```

This tells Nginx to limit requests to the `/wp-login.php` page using the parameters specified in the zone we defined above. Make sure you replace "wordpress" with whatever you named your zone in the previous step. Restart or reload Nginx and rapidly refresh your login page to test if the new brute-force protection is working. If you refresh faster than the rate you defined in `limit_req_zone`, the server will return the status code defined in `limit_req_status`.

**Note:** If you've read other guides on how to set up rate limiting with Nginx, you may have seen other configuration options used, such as `limit_req zone=one burst=1 nodelay`. The `burst` and `nodelay` options are more complex and allow you to control what happens to excess requests. They are not necessary in this context, since we want any excess brute-force attempts to be immediately rejected, but I would highly encourage you to read [the documentation for them](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req).

## Conclusion

This is by no means an exhaustive list for preventing WordPress brute-force attacks. Other solutions exist in the form of WordPress plugins or server-level intrusion prevention systems. However, a lot can be accomplished by correctly configuring Nginx, and the fewer WordPress plugins you have installed, the better.
