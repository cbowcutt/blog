---
layout: post
title: Using the C# HTTPClient to POST a binary file to a server
date: 2018-06-12
category: automation
---
### Using the C# HTTPClient to POST a binary file to a server



##### The HttpClient class

You can call PostAsync() on an instance of HTTPClient to upload binary content to a server. The first parameter specifies the Uri of the request, and it can either be a String or Uri object. The second parameter is of the type HttpContent, and it is the request content sent to the server.

##### The HttpContent class

The HttpContent class is within the System.Net.Http namespace. Instances of this class provide methods for writing HTTP content to a stream or buffer. It also contains an instance of the HttpContent.Headers class, and it can be accessed like a property. However, HttpContent is only a base class, and doesn't provide everything needed to send a POST request with binary data. Instead, the ByteArrayContent class is used.

#####  The ByteArrayContent class

The binary data that we want to send in our POST request will be contained within an instance of ByteArrayContent. Since this class inherits HttpContent, it will also contain headers for the HTTP request. 

```c#
using System.Net.Http;
List<Byte> data = new List<Byte>
... // add stuff to data
HttpContent binaryContent = new ByteArrayContent(data.ToArray());

binaryContent.Headers.Add("Content-Length", 1024)
```

In a POST request, the Content-Type header tells the server the type of data that will be sent. Since we are sending binary data,  the Content-Type header will be assigned a value of "application/octet-stream."

```c#
binaryContent.Headers.Add("Content-Type", "application/octet-stream");
```



##### Sending the data

A string representing the URI can be passed in along with the BinaryContent instance to send the data to the desired server. 

##### Receiving the response

