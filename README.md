---
title: UploadFileEx
grammar_cjkRuby: true
---


> This artical is very useful for those who want to upload files by `winform` program, and receive these files by `php webservice`. I found the artical [here][1] and share to you. Thank for the author.


-------


Main content
------
> UploadFile hides some of the things you might need to get your Windows client simulating forms with file input fields. UploadFileEx gives you more control where you need it!

Introduction
---
Okay, so you like the idea behind WebClient's UploadFile function, but you can't get it to do all the things you want? That's were I was at a week ago. This is what I learned in the past week. UploadFile seems good enough on the surface but you can run into some tricky things fast, like if you have some cookies to manage or you want to specify the content type, or change the name of the file input in the form? Well, this function will take care of those problems and should make simulating a web form with input type=file as easy as pi, er.. pie. Under the hood is a HttpWebRequest version of UploadFile that you could use to add additional functionality.

For example...
-----
You would like to simulate this web form from your C# application:
```c#
<form action ="http://localhost/test.php" method = POST>
<input type = text name = uname>
<input type = password name =passwd>
<input type = FILE name = uploadfile>
<input type=submit>
</form>
```
You could run UploadFileEx like this:
```c#
CookieContainer cookies = new CookieContainer();
//add or use cookies
NameValueCollection querystring = new NameValueCollection();
querystring["uname"]="uname";
querystring["passwd"]="snake3";
string uploadfile;// set to file to upload
uploadfile = "c:\\test.jpg";

//everything except upload file and url can be left blank if needed
string outdata = UploadFileEx(uploadfile, 
     "http://localhost/test.php","uploadfile", "image/pjpeg", 
     querystring,cookies);
```
So on to the code...
----
So, this is UploadFileEx:
```c#
public static string UploadFileEx( string uploadfile, string url, 
    string fileFormName, string contenttype,NameValueCollection querystring, 
    CookieContainer cookies)
{
    if( (fileFormName== null) ||
        (fileFormName.Length ==0))
    {
        fileFormName = "file";
    }

    if( (contenttype== null) ||
        (contenttype.Length ==0))
    {
        contenttype = "application/octet-stream";
    }


    string postdata;
    postdata = "?";
    if (querystring!=null)
    {
        foreach(string key in querystring.Keys)
        {
            postdata+= key +"=" + querystring.Get(key)+"&";
        }
    }
    Uri uri = new Uri(url+postdata);


    string boundary = "----------" + DateTime.Now.Ticks.ToString("x");
    HttpWebRequest webrequest = (HttpWebRequest)WebRequest.Create(uri);
    webrequest.CookieContainer = cookies;
    webrequest.ContentType = "multipart/form-data; boundary=" + boundary;
    webrequest.Method = "POST";


    // Build up the post message header
    StringBuilder sb = new StringBuilder();
    sb.Append("--");
    sb.Append(boundary);
    sb.Append("\r\n");
    sb.Append("Content-Disposition: form-data; name=\"");
    sb.Append(fileFormName);
    sb.Append("\"; filename=\"");
    sb.Append(Path.GetFileName(uploadfile));
    sb.Append("\"");
    sb.Append("\r\n");
    sb.Append("Content-Type: ");
    sb.Append(contenttype);
    sb.Append("\r\n");
    sb.Append("\r\n");            

    string postHeader = sb.ToString();
    byte[] postHeaderBytes = Encoding.UTF8.GetBytes(postHeader);

    // Build the trailing boundary string as a byte array
    // ensuring the boundary appears on a line by itself
    byte[] boundaryBytes = 
           Encoding.ASCII.GetBytes("\r\n--" + boundary + "\r\n");

    FileStream fileStream = new FileStream(uploadfile, 
                                FileMode.Open, FileAccess.Read);
    long length = postHeaderBytes.Length + fileStream.Length + 
                                           boundaryBytes.Length;
    webrequest.ContentLength = length;

    Stream requestStream = webrequest.GetRequestStream();

    // Write out our post header
    requestStream.Write(postHeaderBytes, 0, postHeaderBytes.Length);

    // Write out the file contents
    byte[] buffer = new Byte[checked((uint)Math.Min(4096, 
                             (int)fileStream.Length))];
    int bytesRead = 0;
    while ( (bytesRead = fileStream.Read(buffer, 0, buffer.Length)) != 0 )
        requestStream.Write(buffer, 0, bytesRead);

    // Write out the trailing boundary
    requestStream.Write(boundaryBytes, 0, boundaryBytes.Length);
    WebResponse responce = webrequest.GetResponse();
    Stream s = responce.GetResponseStream();
    StreamReader sr = new StreamReader(s);

    return sr.ReadToEnd();
}
```
This might help too..
----
You can handle the post in various ways but this is the PHP file I wrote to test:
```php
<?php

	print_r($_REQUEST);

	$uploadDir = '%SOMEPATH';
	$uploadFile = $uploadDir . $_FILES['userfile']['name'];
	print "<PRE>";
	if (move_uploaded_file($_FILES['userfile']['tmp_name'], $uploadFile))
	{
		print "File is valid, and was successfully uploaded. ";
	}
	else
	{
		print "Possible file upload attack!  Here's some debugging info:\n";
		print_r($_FILES);
	}
	print "</PRE>";
?>
```

Credits...
-----
I found most of this out by trial and error searching the web. One major source of this information was this article.

License
----
This article has no explicit license attached to it but may contain usage terms in the article text or the download files themselves. If in doubt please contact the author via the discussion board below.
A list of licenses authors might use can be found [here][2]


  [1]: https://www.codeproject.com/Articles/8600/UploadFileEx-C-s-WebClient-UploadFile-with-more-fu
  [2]: https://www.codeproject.com/info/Licenses.aspx
