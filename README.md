# Teknologisk BizTalk Course March 2018

## Table of content
1. [Restart BizTalk](#powershell-restart-biztalk)
5. [Promotion](#promotion)
6. [Pipeline](#pipeline)
7. [Flat Files](#flat-files)
8. [Debatch](#debatch)
9. [Notes](#notes)
## Powershell Restart BizTalk

```powershell
get-service -name "BTS*" | restart-service
```



### Promotion

```xml
<LoanApp xmlns="http">
    <ID>400</ID>
    <Type>Quick</Type>
    <Amount>1000</Amount>
</LoanApp>
```

[Back to top](#table-of-content)

## Pipeline

- Create a new Project (VS) Kursus.Common.Pipelines (Remember to Sign and specify the correct BizTalk Application) (Kursus.Common)
- Create a new Receive Pipeline inside the Project (ReceiveXMLValidate)
- Use the _XMLDisassembler_ and the _XMLValidate_ Pipeline Components in the correct stages (Disassemble, Validate)
- Deploy
- Use the Pipeline (Remember to reference the common Application) in your existing *Receive Location*, make an validating error in your sample xml document (validation error, it still needs to be valid xml).
- Test that your new Pipeline validates and fails when an invalid document is submitted

[Back to top](#table-of-content)

## Flat files

> For each Record you need to ask yourself the following:
- What are the children delimited by?
- Is it *infix* (a,b,c) or *postfix* (a,b,c,)

### Sample

```
1002;MS
OL:400;40
OL:20;60

```

[Back to top](#table-of-content)

1. Create a new Project (Kursus.Pipeline) (Remember to sign and set BizTalk application)
2. Create a sample document (order.txt) from the sample above
3. Create a new Flat file schema inside your new Project
4. Create the following structure:
```
     Order (Postfix, Hex delimiter 0x0D 0x0A)
       OrderHeader (Infix, Char delimiter ;)
            OrderID
            Customer
        OrderLine (Infix, Char delimiter ;  Tag Identifier: OL:)
            ItemNo
            Quantity
```
5. Validate your sample document with the schema (see that you get correct xml)
6. Create a new Receive Pipeline (ReceiveFFOrder)
7. Use the **FFDisassembler**
8. Try to build, you will get an error
9. Set the document Schema on the FFDisassembler to the FF schema you created before.
10. Deploy
11. Create a new *Receive Location* on the same Receive Port you have used before (make sure that you have a *Send Port* that subscribes to all messages from the Receive Port). ([Name of your ReceivePort].FFOrder) Mask: *.txt
12. Use your new Pipeline on the new *Receive Location*
13. Submit a FF order through the new Receive Location. See that you get an *XML* document in your *Send Port*

[Back to top](#table-of-content)


## Debatch

```xml

<Batch xmlns="http://dti.dk">
<LoanApp xmlns="http://firma">
	<ID>1004</ID>
	<Customer>
		<Name>Morten</Name>
	</Customer>
	<Type>Quick</Type>
</LoanApp>
<LoanApp xmlns="http://firma">
	<ID>1004</ID>
	<Customer>
		<Name>Morten</Name>
	</Customer>
	<Type>Quick</Type>
</LoanApp>
<LoanApp xmlns="http://firma">
	<ID>1004</ID>
	<Customer>
		<Name>Morten</Name>
	</Customer>
	<Type>Quick</Type>
</LoanApp>
</Batch>

```

1. Make a new Project (Kursus.Debatch) and do what is needed!!!
2. Make a new Schema with root element: Batch and namespace http://dti.dk
3. set Envelope to yes (Click on <Schema> and choose Envelope from Properties)
4. Set Body X-Path to Batch (Click on root element, and select **Body XPath** choose Batch and click ok)
5. Deploy Schema
6. Make sure that the actual messages inside the Batch message is valid messages according to your schema(s)
7. Submit the batch and verify that several messages are send to your Send Port
8. Make one of the messages invalid (change the namespace)
9. Submit again
10. Verify that the whole batch is suspended
11. Change the Receive Pipeline properties on the *Receive Location* (RecoverableInterchangeProcessing = True)
12. Submit again and verify that you now get two messages out and one suspended message

[Back to top](#table-of-content)

## Notes

### Namespaces

**Default namespace**
> Namespace is applied to all elements where nothing else is defined

```xml
<Order xmlns="http://namespace">
	<ID>1000</ID>
</Order>

```

In this example both Order and ID gets the namespace applied

**Namespace prefix**
> Namespace is applied only to elements where the prefix is applied

```xml
<aa:Order xmlns:aa="http://namespace">
	<ID>1000</ID>
</aa:Order>

```

In this example only Order is given the namespace applied

#### Examples

1.

```xml

<Order xmlns="http">
	<ID>14</ID>
	<Desc>Morten</Desc>
</Order>

```

2.
```xml

<aa:Order xmlns:aa="http">
	<ID>14</ID>
	<Desc>Morten</Desc>
</aa:Order>


```

3.
```xml

<aa:Order xmlns:aa="http">
	<aa:ID>14</aa:ID>
	<aa:Desc>Morten</aa:Desc>
</aa:Order>


```

4.
```xml

<Order xmlns="http">
	<ID xmlns="">14</ID>
	<Desc xmlns="">Morten</Desc>
</Order>

```


5. 

```xml
<aa:Order xmlns:aa="http" xmlns:bb="whatever">
	<aa:ID>14</aa:ID>
	<aa:Desc>Morten</aa:Desc>
</aa:Order>

```


> Document 1, 3 and 5 are identical

> Document 2 and 4 are identical

[Back to top](#table-of-content)


### Custom XSLT example

```xml

<?xml version="1.0" encoding="UTF-16"?>
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" 
                xmlns:var="http://schemas.microsoft.com/BizTalk/2003/var" exclude-result-prefixes="msxsl var cust userCSharp" 
                version="1.0" 
                xmlns:cust="http://customer.dk/schema" 
                xmlns:int="http://dti.dk/schemas/v10" 
                xmlns:userCSharp="http://schemas.microsoft.com/BizTalk/2003/userCSharp">
  <xsl:output omit-xml-declaration="yes" method="xml" version="1.0" />
  <xsl:template match="/">
    <xsl:apply-templates select="/cust:Order" />
  </xsl:template>
  <xsl:template match="/cust:Order">
    <int:Order>
      <int:ID>
        <xsl:value-of select="OrderID" />
      </int:ID>
      <int:Customer>
        <xsl:text>DS</xsl:text>
      </int:Customer>
      <int:OrderDate>
        <xsl:value-of select="userCSharp:DateCurrentDate()"/>
      </int:OrderDate>
      <int:Desc>
        <xsl:value-of select="Description"/>
      </int:Desc>
      <xsl:for-each select="OrderLine">
        <int:Line>
          <xsl:attribute name="no">
            <xsl:value-of select="position()" />
          </xsl:attribute>
          <int:ItemNo>
            <xsl:value-of select="ItemNumber" />
          </int:ItemNo>
          <int:Qty>
            <xsl:value-of select="Quantity"/>
          </int:Qty>
          <!--"Heavy" if Quantity is greater than 1000, otherwise "Not heavy"-->
          <int:Desc>
            <xsl:choose>
              <xsl:when test="Quantity > 1000">
                <xsl:text>Heavy</xsl:text>
              </xsl:when>
              <xsl:otherwise>
                <xsl:text>Not heavy</xsl:text>
              </xsl:otherwise>
            </xsl:choose>
          </int:Desc>
        </int:Line>
      </xsl:for-each>
    </int:Order>
  </xsl:template>
  <msxsl:script language="C#" implements-prefix="userCSharp"><![CDATA[



public string DateCurrentDate()
{
	DateTime dt = DateTime.Now;
	return dt.ToString("yyyy-MM-dd", System.Globalization.CultureInfo.InvariantCulture);
}


]]></msxsl:script>
</xsl:stylesheet>

```

### Generate Schema from sample XML

> Make sure that you run *InstallWFX.vbs* once on each developer machine

```powershell

C:\Program Files (x86)\Microsoft BizTalk Server 2016\sdk\Utilities\Schema Generator\InstallWFX.vbs

```

Right click on BizTalk Project (VS)

Add Generated Items>Generate Schemas>Well Formed XML 

