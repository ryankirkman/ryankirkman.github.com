---
layout: post
title: Pass Through Cross Domain Proxies with jQuery
---

# {{page.title}}

If you're building a client-side javascript application that uses external API's you will no doubt have come into contact with the dreaded CORS [Cross Origin Resource Sharing](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing).

CORS means that unless the API server specifically allows it, you won't be able to make a request to an API on a domain different to that of the calling code. For example, you wouldn't be able to call *http://exampleapi.com/getdata* from javascript served from *http://example.com/* if CORS wasn't enabled on *http://exampleapi.com/*.

There are two ways around this. The first, and easiest assuming you control the API server is to enable CORS. I'll leave that as an exercise for the reader.

The second is to create a pass through proxy that allows you to make cross domain requests from your current domain. I've coded up an example for you in C#, but it's so simple that you could port it to almost any language used in web development.

``` csharp
// ProxyRequest proxies a request to a JSON-based API
// to avoid the cross origin request issue.
// It assumes the API supports POST.
// JsonResult is an ASP.NET MVC construct.
private JsonResult ProxyRequest(string url, string data)
{
	HttpWebRequest wr = (HttpWebRequest)HttpWebRequest.Create(url);
	wr.Method = "POST";
	wr.ContentType = "application/json";
 
	// Set the data to send.
	using (var streamWriter = new StreamWriter(wr.GetRequestStream()))
	{
		streamWriter.Write(data);
	}
 
	// Get the response.
	var httpResponse = (HttpWebResponse)wr.GetResponse();
	using (var streamReader = new StreamReader(httpResponse.GetResponseStream()))
	{
		JsonResult jr = new JsonResult();
		jr.Data = streamReader.ReadToEnd();
		return jr;
	}
}
```

How do you use this with jQuery?

``` javascript
// See: http://api.jquery.com/jQuery.ajaxPrefilter/
$.ajaxPrefilter( function( options ) {
  if ( options.crossDomain ) {
    var newData = {};
    // Copy the options.data object to the newData.data property.
    // We need to do this because javascript doesn't deep-copy variables by default.
    newData.data = $.extend({}, options.data);
    newData.url = options.url;
 
    // Reset the options object - we'll re-populate in the following lines.
    options = {};
 
    // Set the proxy URL
    options.url = "http://mydomain.com/proxy";
    options.data = $.param(newData);
    options.crossDomain = false;
  }
});
 
// How to use the cross domain proxy
$.ajax({
  url: 'http://the-real-api-url.com/getdata',
  data: {
    username: 'myUsername',
    password: 'myPassword'
  },
  crossDomain: true, // set this to ensure our $.ajaxPrefilter hook fires
  processData: false // We want this to remain an object for  $.ajaxPrefilter, and for performance reasons
}).success(function(data) { // Use the new jQuery promises interface and assume our API call returns a JSON object
    var jsonData = JSON.parse(data); // Assume it return a JSON string
    console.log(jsonData); // Do whatever you want with the data
});
```