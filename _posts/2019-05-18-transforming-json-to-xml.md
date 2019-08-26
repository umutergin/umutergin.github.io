---
title: A Study in Alchemy - Transforming JSON Requests to XML for Exploitation(Google CTF)
---

When I was playing with Google ctf challenges I have come across a rather simple but unorthodox challenge that required some good old out of the box thinking. Upon entering the challenge page, you're welcomed by a message intended for blind people using the braille alphabet. Also, there is a dropdown menu which you can select 3 different cities and a submit button.

![image](/img/webpage.png)

When you intercept the traffic and take a look at the submit request, it is observed to be sent to /api/search endpoint with a message using JSON format. Message contains some number according to your choosing of city. Request is shown below;

![image](/img/request1.png)

After trying all three cities, it is found that;

For Zurich
{"message":"135601360123502401401250"}
For Bangalore
{"message":"120101345012450101230135012350150"}
For Paris
{"message":"1234010123502402340"}

Since the web page contained a message in braille alphabet, I tried to decipher messages according to [braille.](https://www.pharmabraille.com/pharmaceutical-braille/the-braille-alphabet/)

![image](/img/braille-ss3.png)

z is 1356
u is 136
r is 1235
i is 24
c is 14
h is 1235

As you can deduce, letter numbers are joined with 0, forming the corresponding cities. After this information, I have tried to post various messages including "flag", "google" etc, but it did not get me any succesful results. I have also tried other vulnerabilities like SQL injection but again, they failed me and I finally got to subject of this post.

When it comes to APIs, it is a big debate among the web developers to use XML or JSON as data formats. It was clear that this API was using JSON format for transmitting data. After thinking on the subject I remembered an [article](https://api2cart.com/api-technology/json-vs-xml-api-data-interaction/) about XML vs JSON subject and it reminded me of something, why not both?

I have tried to transmit request using XML format by changing the Content-type: application/json to Content-type:application/xml. It resulted in a syntax error, as expected since I did not form a valid XML but it did reveal that this API parses XML. You can use a BurpSuite plugin called Content Type Convertor for this, I used manual editing.

![image](/img/xml-ss4.png)

It is best to transform original JSON request to XML version, then it is easier to fiddle for XXE.

![image](/img/jsonToxml-ss8.png)

At this point, I am aware that application is parsing XML, knowing this, I have tried various XXE methods using external DTD files but none of them gave any result. Then I decided to try loading internal DTDs. To be able to exploit an internal DTD file, there is an awesome tutorial on the [Web Security Academy](https://portswigger.net/web-security/xxe/blind/lab-xxe-trigger-error-message-by-repurposing-local-dtd) page. On linux systems, DTD file is present under directory /usr/share/yelp/dtd/ and it contains several entities including ISOamsa, ISOamsb, etc. 

![image](/img/entities-isoamsa-ss7.png)

This technique works by referencing a local DTD file that is present on the linux hosts(dockbookx.dtd) and then redefining entity called [isoamso](https://www.oreilly.com/openbook/docbook/book/iso-amso.html) which is a parameter entity that includes ISO characters and math symbols to call the /etc/passwd file. In this example I have invoked isoamsa entity but any other entity could be used as well.

Here are the exploiting steps:

- First, reference an internal DTD file that is present on the host like docbookx.dtd.
- Invoke the file that is to be read, which is /etc/passwd in this case. 
- Then, read the contents of that file with "eval" and reference it in XML(named error) which will result in error because there is no such file(nonexistent).
- Last, application will throw an error like "There is no such file as 'root:x:0:0:x'" because the content of the file is the file name that I tried to read.
- Content of original file will be included in error message.

![image](/img/xxe1-ss5.png)

For the last part, once we edit the /etc/file as /flag, we are welcome with the flag.

![image](/img/xxe2-ss6.png)
