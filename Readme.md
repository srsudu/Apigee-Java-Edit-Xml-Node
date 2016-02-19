# Java Add Xml Node

This directory contains the Java source code and pom.xml file required to
compile a simple custom policy for Apigee Edge. The policy adds a node to an XML document. 

Suppose you have a document like this: 
```xml
 <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"> 
  <soap:Header/> 
  <soap:Body> 
    <act:test xmlns:act="http://yyyy.com"> 
      <abc> 
        <act:demo>fokyCS2jrkE5s+bC25L1Aax5sK//FkYA1msxIyW7prOun0VwoSET73UXKyKJ7nmd3OwHq/08GXIpwlq3QBJuG7a4Xgm4Vk</act:demo> 
      </abc> 
    </act:test> 
  </soap:Body> 
</soap:Envelope> 
```

And you'd like to replace the text node in the middle of that document with something else, maybe the decrypted version of that string. This policy lets you do that, allowing you to transform the above into this: 

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"> 
  <soap:Header/> 
  <soap:Body> 
    <act:test xmlns:act="http://yyyy.com"> 
      <abc> 
        <act:demo>decrypted-text-here</act:demo> 
      </abc> 
    </act:test> 
  </soap:Body> 
</soap:Envelope>
```

No coding required! 


## Using this policy

You do not need to build the source code in order to use the policy in Apigee Edge. 
All you need is the built JAR, and the appropriate configuration for the policy. 
If you want to build it, feel free.  The instructions are at the bottom of this readme. 


1. copy the jar file, available in  target/edge-custom-add-xml-node.jar , if you have built the jar, or in [the repo](bundle/apiproxy/resources/java/edge-custom-add-xml-node.jar) if you have not, to your apiproxy/resources/java directory.  You can do this offline, or using the graphical Proxy Editor in the Apigee Edge Admin Portal. 

2. include an XML file for the Java callout policy in your
   apiproxy/resources/policies directory. It should look
   like this:  
   ```xml
    <JavaCallout name='Java-AddXmlNode-1'>
        ...
      <ClassName>com.dinochiesa.edgecallouts.AddXmlNode</ClassName>
      <ResourceURL>java://edge-custom-add-xml-node.jar</ResourceURL>
    </JavaCallout>
   ```  

3. use the Edge UI, or a command-line tool like pushapi (See
   https://github.com/carloseberhardt/apiploy) or similar to
   import the proxy into an Edge organization, and then deploy the proxy . 
   Eg,    
   ```./pushapi -v -d -o ORGNAME -e test -n add-xml-node ```b

4. Use a client to generate and send http requests to the proxy you just deployed . Eg,   
   ```
curl -i -X POST -H content-type:text/xml \
  'http://ORGNAME-test.apigee.net/add-xml-node/t1?texttoadd=seven&xpath=/root/a/text()' \
  -d '<root><a>beta</a></root>'
```


## Notes on Usage

There is one callout class, com.dinochiesa.edgecallouts.AddXmlNode.

The policy is configured via properties set in the XML.  You can set these properties: 

* xpath - the xpath to resolve to a single node in the source document
* source - the source xml document. This should be a context variable. Specifying this is optional.  If you omit this property, the policy will use "message.content" as the source. 
* new-node-type - should be one of element, attribute, text.
* new-node-text - Depending on the value of new-node-type, this must take a value that corresponds to an element, attribute, or text node.  For an element, eg, <foo>bar</foo>.  For an attribute, do not use any quotes.  Eg, attr1=value.  Or, for a Text node, any text string. 
* action - append, insert-before, or replace
* output-variable - the name of a variable to hold the result. Optional. If not present, the result is placed into "message.content". 


## Example Policy Configurations

### Appending an Element

```xml
<JavaCallout name='Java-AddXmlNode-1'>
  <Properties>
    <Property name='source'>request.content</Property>
    <Property name='new-node-type'>element</Property>
    <Property name='new-node-text'><Foo>text-value</Foo></Property>
    <Property name='xpath'>{request.queryparam.xpath}</Property>
    <Property name='action'>append</Property>
  </Properties>
  <ClassName>com.dinochiesa.edgecallouts.AddXmlNode</ClassName>
  <ResourceURL>java://edge-custom-add-xml-node.jar</ResourceURL>
</JavaCallout>
```

### Replacing a text node

```xml
<JavaCallout name='Java-AddXmlNode-1'>
  <Properties>
    <Property name='source'>request.content</Property>
    <Property name='new-node-type'>text</Property>
    <Property name='new-node-text'>{request.queryparam.texttoinsert}</Property>
    <Property name='xpath'>{request.queryparam.xpath}</Property>
    <Property name='action'>replace</Property>
    <Property name='output-variable'>my_variable</Property>
  </Properties>
  <ClassName>com.dinochiesa.edgecallouts.AddXmlNode</ClassName>
  <ResourceURL>java://edge-custom-add-xml-node.jar</ResourceURL>
</JavaCallout>
```

### Replacing a text node using XML namespaces

```xml
<JavaCallout name='Java-AddXmlNode-2'>
  <Properties>
    <Property name='xmlns:soap'>http://schemas.xmlsoap.org/soap/envelope/</Property>
    <Property name='xmlns:act'>http://yyyy.com</Property>
    <Property name='source'>request.content</Property>
    <Property name='new-node-type'>text</Property>
    <Property name='new-node-text'>{request.queryparam.texttoinsert}</Property>
    <Property name='xpath'>/soap:Envelope/soap:Body/act:test/abc/act:demo/text()</Property>
    <Property name='action'>replace</Property>
  </Properties>
  <ClassName>com.dinochiesa.edgecallouts.AddXmlNode</ClassName>
  <ResourceURL>java://edge-custom-add-xml-node.jar</ResourceURL>
</JavaCallout>
```




## Building:

Requires Java 1.7, and Maven. 


1. unpack (if you can read this, you've already done that).

2. configure the build on your machine by loading the Apigee jars into your local cache
  ```./buildsetup.sh```

2. Build with maven.  
  ```mvn clean package```

This will run tests.



## Build Dependencies

- Apigee Edge expressions v1.0
- Apigee Edge message-flow v1.0
- codehaus jackson 1.9.7

These jars must be available on the classpath for the compile to
succeed. The buildsetup.sh script will download the Apigee files for
you automatically, and will insert them into your maven cache.  The pom file will take care of the other Jar. 

If you want to download them manually: 

The first two jars are
produced by Apigee; contact Apigee support to obtain these jars to allow
the compile, or get them here: 
https://github.com/apigee/api-platform-samples/tree/master/doc-samples/java-cookbook/lib

The other can be downloaded from Maven central. 



## Bugs

None known.