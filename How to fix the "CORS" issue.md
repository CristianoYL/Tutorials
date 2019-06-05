# **Introduction**

> Access to fetch at 'https://www.yourapi.com' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

If you have ever built a web app that wants to interact with a REST API, you might be familiar with this error. If you also built the API, you might have wondered: why did it work with Postman when you were testing your API, but not in the browser? *There must be something wrong with my web app.* However, this is probably not the case.

This article will give you an overall idea of what CORS is, and how to '*fix*' it.

# **CORS**

## **What is CORS**

**CORS** stands for Cross-Origin Resource Sharing. It's a mechanism that restricts requests coming from a different origin (domain). A request coming from a different origin is known as a cross-origin request. Cross-origin requests are vital for when your site needs to load data from other services.

CORS allows servers to specify who can access their resources and how. Browsers follow the servers' policies by sending a *test* request (preflight) to the server and checking whether it's allowed. I'll go into a bit more detail in the following sections.

## **Why CORS**

The rationale behind it is to allow the server (API) on one origin to restrict behaviour for other origins, since one may only want to allow others to read the data, but not to modify the data at will. As a result, requests like **GET** are usually allowed by default. However, **PUT**, **DELETE** and sometimes **POST** would be restricted.

## **How does CORS work**

When the browser is about to send a request that will trigger CORS to a different origin, e.g. a **PUT** request, it will not send the actual request immediately. Instead, it will send what's called a *preflight* request, which serves as a *test* as to whether the server will allow the communication. If the server permits, in response to the preflight request, the browser will then send the actual request. Otherwise, it will reject the request, returning a 405 status code and the error message shown at the start of this post.

## **References**

So far, I've provided a very brief explanation of CORS. For those who wants to dig deeper, you can find a more thorough explanation of CORS on the MDN: [https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

Another good read would be [What is CORS?](https://www.codecademy.com/articles/what-is-cors)

# **How to make it work**

The CORS issue can be very annoying, especially for learners. This article will sum up the solutions which may save your hours of searching on StackOverflow. The solution depends on your particular scenario: more specifically, on how much control you have over the server. We will go from the *best case* scenario to the *worst case*.

## **Scenario 1: full control over the server**

If you are the one that deploys the API onto your own server, then you are in the best case scenario. All you need to do is to configure your server, assuming that you're using **Nginx**. If you're not, you can still check out the following scenarios which may also fix this issue.

Here's a code snippet from [https://enable-cors.org/server_nginx.html](https://enable-cors.org/server_nginx.html) which shows how to configure a wide-open CORS policy.

    if ($request_method = 'OPTIONS') {
      add_header 'Access-Control-Allow-Origin' '*';
      add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
    
      # Custom headers and headers various browsers *should* be OK with but aren't
      add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
    
      # Tell client that this pre-flight info is valid for 20 days
      add_header 'Access-Control-Max-Age' 1728000;
      add_header 'Content-Type' 'text/plain; charset=utf-8';
      add_header 'Content-Length' 0;
      return 204;
    }
    if ($request_method = 'POST') {
      add_header 'Access-Control-Allow-Origin' '*';
      add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
      add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
      add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
    }

You can tailor it to your needs. For example, instead of using a wildcard for all origins, you can permit only your web app domain. You can also choose to only allow specific methods and headers to go through.

This solution allows you to configure CORS without tampering with your API code, which is ideal in most cases. If this is not want you want, or you do not have control over the server, you may find your solution in the next scenario. For example if you deployed your API via **Heroku**.

# **Scenario 2: my API but not my server**

In this scenario, you fully deployed your API, but you might be using a hosting service like **Heroku** so you do not have control over the server configuration. This section guides you through adding the necessary header in your backend code.

There is no universal server-side language or framework, but we can at least show a solution using one very popular option.

Here I'm going to use a **Flask** app for demonstration, which is a very popular **Python** framework (sorry **Django** lovers!).

    app = Flask(__name__)
    
    @app.after_request
    def after_request(response):
        response.headers.add('Access-Control-Allow-Origin', '*')
        response.headers.add('Access-Control-Allow-Headers', 'Content-Type')
        return response

The above code block shows how to add the additional header after the request has been handled and before the response is sent. The same applies here as with Nginx: you can tailor the solution to be less open, and only allow certain origins, methods, or headers.

# **Scenario 3: not my API and not my server**

Front-end developers might be in the 'worst' scenario here. However, it's not your fault and it's quite common situation to find yourself in. You might just want to interact with a third party API and it just keeps complaining about CORS. What can we do about this?

The ideal solution would be inform the API owner about this issue, since technically this is **their fault**. However, you might not get an immediate response. Since they found no issue with using it, it's your own misconfiguration. Please refer them to this post if this is your situation. However, in the meantime you could try the following solution.

You can build a proxy API that serves as a bridge to the third party API. It sounds a bit hacky but it might be the only working solution you available if you wish your web app to work. The proxy API would receive the requests from your web app and send them to the third party API. When the third party API responds, it would just forward the response to your web app. Since the requests to the third party API don't go through the browser, it circumvents the CORS protection. You can then configure your own proxy API as in the previous two scenarios.

The last solution would be using browser plugins to inject the needed headers into the response. This is not recommended since it's only 'fooling yourself'. Browser plugins would not work on other computers (such as your users') unless they have the plugins too. However, it is still useful if you're testing things around locally and would like to see some results ASAP. There are usually plenty of such plugins for each browser. One of them is the [Allow-Control-Allow-Origin](https://chrome.google.com/webstore/detail/allow-control-allow-origi/nlfbmbojpeacfghkpbjhddihlkkiljbi) extension for Chrome.

There are also tutorials on how to turn off the CORS check on browsers, but we won't not cover that here since it's not scalable either.

# **Conclusion**

This post provided a very high-level view on the CORS policy. There're a lot more to learn, which can be found in the references section at the top. I've tailored the solutions to different scenarios to help you find the solution that suits your needs. I hope you find this article helpful, since I've personally encountered these scenarios and wished for an article like this one. Cheers!