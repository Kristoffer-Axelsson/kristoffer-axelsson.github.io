## Part 2: Generating queryable .odc files for consumeable OData feeds

## How to serve the feed?
Well, now core code up. But the business case here is that the user should be able to download the file through the web application similar to Microsoft and many other services. How can we achieve that? How can we deliver the feed in a way that the user can easily open and consume?

The requirements for delivering the feed at this stage are:
- It should be possible to consume the feed with minimal effort from the web portal
- It should not be a static Excel export, it needs to be queryable, this is the biggest point.
- It should be a solution can that be opnened in Excel, preferably with no additional login, if the session is still active.

How can we serve this to the user?

## Odc
.Odc is an open, xml-based, Excel compatible file format. It uses the OpenDocument standard established by OASIS (Organization for the Advancement of Structured Information Standards), just like OData itself. In this case, it's goning to be used basically like a small db file containing a data soruce url and additional information like headers etc. During the development phase of this solution I worked closely with a development team in an other part of Europe, taking a more tech lead, directive role together with the client. Even though we were a larger team at this point, it became noticable just how scarce the documentation is on this file format. More infromation can be found here https://www.odata.org/documentation/

Hopefully this information will help others trying to create consumable OData feeds.

Solution:
Odc is just part of the solution. Below are the solutions steps, or flow if you will, that I drew up in my head after disussions with the client to fulfill the business need. 

1. The user signs into the SPA application.
2. The user navigates to one of the pages, i.e. Orders.
3. The user performs eventual filtering until the user is pleased.
4. The user clicks an Excel icon
5. A request is made to one of the backend API operations to generate an odc file with dynamic data, i.e. the filtering.
6. The odc file is served to the SPA and downloaded in the user's browser.
7. When the user clicks the file, it should automatically open in Excel if it's installed.
8. When the user opens Excel the OData feed should trigger an authentication handshake toward the authentication source.
9. The data is served to Excel from the OData feed data source, in this case the OData operation built in Part 1, and the user can browse, query and analyze the data as wanted.

## Creating an Odc file
Generating an odc file proved more difficult than expected, not because of technical difficulty, but because lack of documentation. In March 2020, a search on google for "odc file odata" returns only a mere 42600 results. However, download a few files from PowerBi gave a starting point in constructing an odc file.

I will share the odc template we used, hopefully this can be engineers out there a head start when developing OData services:
<html xmlns:o="urn:schemas-microsoft-com:office:office" xmlns="http://www.w3.org/TR/REC-html40">
<head>
<meta http-equiv=Content-Type content="text/x-ms-odc; charset=utf-8">
<meta name=ProgId content=ODC.Database>
<meta name=SourceType content=OLEDB>
<title>Query - GetOrderQuery</title>
<xml id=docprops><o:DocumentProperties xmlns:o="urn:schemas-microsoft-com:office:office" xmlns="http://www.w3.org/TR/REC-html40">
  <o:Description>Connection to the 'GetOrderQuery' query in the workbook.</o:Description>
  <o:Name>Query - GetOrderQuery</o:Name>
 </o:DocumentProperties>
</xml><xml id=msodc><odc:OfficeDataConnection xmlns:odc="urn:schemas-microsoft-com:office:odc" xmlns="http://www.w3.org/TR/REC-html40">
  <odc:PowerQueryConnection odc:Type="OLEDB">
   <odc:ConnectionString>Provider=Microsoft.Mashup.OleDb.1;Integrated Security=ClaimsToken;Data Source=$Workbook$;Location=GetOrderQuery;Extended Properties=&quot;&quot;</odc:ConnectionString>
   <odc:CommandType>SQL</odc:CommandType>
   <odc:CommandText>SELECT * FROM [GetOrderQuery]</odc:CommandText>
  </odc:PowerQueryConnection>
  <odc:PowerQueryMashupData>&lt;Mashup xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot; xmlns:xsd=&quot;http://www.w3.org/2001/XMLSchema&quot; xmlns=&quot;http://schemas.microsoft.com/DataMashup&quot;&gt;&lt;Client&gt;EXCEL&lt;/Client&gt;&lt;Version&gt;2.61.5192.1301&lt;/Version&gt;&lt;MinVersion&gt;2.21.0.0&lt;/MinVersion&gt;&lt;Culture&gt;pl-PL&lt;/Culture&gt;&lt;SafeCombine&gt;true&lt;/SafeCombine&gt;&lt;Items&gt;&lt;Query Name=&quot;GetOrderQuery&quot;&gt;&lt;Description /&gt;&lt;Formula&gt;&lt;![CDATA[let&#13;&#10;    Source = OData.Feed(&quot;https://localhost:44378/odata/user&quot;, null, [Implementation=&quot;2.0&quot;, Query=[#&quot;api-version&quot;=&quot;1.0&quot;]]),&#13;&#10;    #&quot;Filtered Rows&quot; = Table.SelectRows(Source, each ([userName] &lt;&gt; null))&#13;&#10;in&#13;&#10;    #&quot;Filtered Rows&quot;]]&gt;&lt;/Formula&gt;&lt;IsParameterQuery xsi:nil=&quot;true&quot; /&gt;&lt;IsDirectQuery xsi:nil=&quot;true&quot; /&gt;&lt;/Query&gt;&lt;/Items&gt;&lt;/Mashup&gt;</odc:PowerQueryMashupData>
 </odc:OfficeDataConnection>
</xml>
</head>
</html>

The decoded version looks like this:
<html xmlns:o="urn:schemas-microsoft-com:office:office" xmlns="http://www.w3.org/TR/REC-html40">
<head>
<meta http-equiv=Content-Type content="text/x-ms-odc; charset=utf-8">
<meta name=ProgId content=ODC.Database>
<meta name=SourceType content=OLEDB>
<title>Query - GetOrderQuery</title>
<xml id=docprops><o:DocumentProperties xmlns:o="urn:schemas-microsoft-com:office:office" xmlns="http://www.w3.org/TR/REC-html40">
  <o:Description>Connection to the 'GetOrderQuery' query in the workbook.</o:Description>
  <o:Name>Query - GetOrderQuery</o:Name>
 </o:DocumentProperties>
</xml><xml id=msodc><odc:OfficeDataConnection xmlns:odc="urn:schemas-microsoft-com:office:odc" xmlns="http://www.w3.org/TR/REC-html40">
  <odc:PowerQueryConnection odc:Type="OLEDB">
   <odc:ConnectionString>Provider=Microsoft.Mashup.OleDb.1;Integrated Security=ClaimsToken;Data Source=$Workbook$;Location=GetOrderQuery;Extended Properties=""</odc:ConnectionString>
   <odc:CommandType>SQL</odc:CommandType>
   <odc:CommandText>SELECT * FROM [GetOrderQuery]</odc:CommandText>
  </odc:PowerQueryConnection>
  <odc:PowerQueryMashupData><Mashup xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/DataMashup"><Client>EXCEL</Client><Version>2.61.5192.1301</Version><MinVersion>2.21.0.0</MinVersion><Culture>pl-PL</Culture><SafeCombine>true</SafeCombine><Items><Query Name="GetOrderQuery"><Description /><Formula><![CDATA[let
    Source = OData.Feed("https://localhost:44378/odata/deliveries", null, [Implementation="2.0", Query=[#"api-version"="1.0"]]),
    #"Filtered Rows" = Table.SelectRows(Source, each ([userName] <> null))
in
    #"Filtered Rows"]]></Formula><IsParameterQuery xsi:nil="true" /><IsDirectQuery xsi:nil="true" /></Query></Items></Mashup></odc:PowerQueryMashupData>
 </odc:OfficeDataConnection>
</xml>
</head>
</html>

This might be hard to read at first glance, but the file is basically a PowerQuery filter and connection with a SQL command type for filtering. The source of the OData feed here is operation of the OData endpoint. Note that this is just the template though, it needs to be modified. We pass in the filter.

## Creating and serving the actual Odata file
First we start with the API controller and operation, this, like the others, makes a check against the extracted customerId of the incoming access token.

![image](https://kristofferaxelsson.blob.core.windows.net/$web/github/odata_generation1.png)

We generate the filename of the odc based on the type of the data (in this case there's two types, Delivery Tickets and Loads) and make a call to a method to actually generate the byte array that will be the file content. (Anyone who has worked with i.e. pdfsharp or similar libraries will be familiar with the overall procedure here)

![image](https://kristofferaxelsson.blob.core.windows.net/$web/github/odata_generation2.png)

We then generate the file content. The content is based on the template file content where the PowerQuery section is replaced with the dynamic content.

![image](https://kristofferaxelsson.blob.core.windows.net/$web/github/odata_generation3.png)

![image](https://kristofferaxelsson.blob.core.windows.net/$web/github/odata_generation4.png)

We now have a file ready to be served!

## Next
Generating the odc is great, but some where important steps are left regarding this solution, especially regarding securing OData feeds and serving them through Azure API Management. More regarding this is coming up in Part 3.
