# Teknologisk BizTalk Course March 2018

## Table of content
1. [Restart BizTalk](#powershell-restart-biztalk)
5. [Promotion](#promotion)
6. [Pipeline](#pipeline)
7. [Flat Files](#flat-files)
8. [Debatch](#debatch)
9. [Notes](#notes)
10. [Orchestrations](#orchestrations)
11. [Custom Pipeline Components](#custom-pipeline-components)
12. [Final Test](#final-test)

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

#### Tracked promoted properties

1. Select the *Property Schema* under *Schemas*
2. Right-click and choose **Properties** and choose *Tracking*
3. Select the desired properties
4. Make sure that *Tracked Message Properties* is selected on the relevant Port(s)


Tracking:
1. In the **Group Hub** select *Tracked Message Events*
2. Select the *Schema name* (This is **NOT** the Property Schema but the actual schema we are tracking)
3. Select the desired Tracked Property and insert the value needed for tracking



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

Also if a *Tag Identifier* is present it needs to be specified


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

## Orchestrations

1. Create new Project (Kursus.Orc)
2. Create new Schema

```xml
<aa:LoanApp xmlns:aa="orc">
	<ID>10</ID>
	<Customer>Morten</Customer>
	<Amount>1000</Amount>
	<Status>New</Status>
</aa:LoanApp>

```

3. Create new Orchestration (ProcessLoanApp)
4. Under *Orchestration View* create new Message (msgLoanApp)
5. Select Schema as *Message Type*
6. Create new Receive Shape
7. Set Message = msgLoanApp
8. Right click on Port surface select *New Configured Port* (LPort_ReceiveLoanApp)
9. Set *Port Type name* to pt_LoanApp 
10. Next, next, finish
11. Drag the green arrow to the shape
12. Set *Activate* on Receive Shape to True
13. Test that your Project can build
14. Create an Expression Shape and insert:


```powershell
System.Diagnostics.EventLog.WriteEntry("TheOrc","Hello World!");

```

15. Deploy
16. Start your Orchestration (Remember to bind it -> Fill in the blanks)
17. Submit a LoanApp and see that your Orc is writing to the Event Log!



More stuff:

1. Create two folders *ERP* and *Manual*
2. Create two file Send Ports one for each folders
3. Make *Amount* a *Distinguished field* and set the type to *xs:decimal*
4. build the project
5. Create a **Decide** shape under the **Expression** shape
6. Insert the rule msgLoanApp.Amount < 1000 into the left side
7. Create two Logical Send Ports one for ERP and one for Manual*
8. Insert one Send Shape into each Decide branch and connect them (remember to choose msgLoanApp as *Message* on each *Send* shape





[Back to top](#table-of-content)

## Custom Pipeline Components


References:


- Microsoft.BizTalk.Pipeline
- Microsoft.XLANGs.BaseTypes
- System.Drawing

### Code examples

```csharp

using Microsoft.BizTalk.Component.Interop;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Microsoft.BizTalk.Message.Interop;
using System.Collections;
using System.Reflection;
using Kursus.PipelineComponents.Helpers;

namespace Kursus.PipelineComponents
{
    [System.Runtime.InteropServices.Guid("8dc0ddd4-56ca-443f-af56-f0a28d20301f")]
    [ComponentCategory(CategoryTypes.CATID_PipelineComponent)]
    [ComponentCategory(CategoryTypes.CATID_Any)]
    public class AppendOrRemoveCRLF : Microsoft.BizTalk.Component.Interop.IComponent,
      IBaseComponent,
      IPersistPropertyBag,
      IComponentUI
    {
        private System.Resources.ResourceManager resourceManager = new System.Resources.ResourceManager("Vertica.BizTalk.PipelineComponents", Assembly.GetExecutingAssembly());
        protected System.Resources.ResourceManager getRS()
        {
            return resourceManager;
        }

        //Properties
        public bool Append { get; set; }



        public string Description
        {
            get
            {
                return "This is a cool component";
            }
        }

        public IntPtr Icon
        {
            get { return ((System.Drawing.Bitmap)(getRS().GetObject("COMPONENTICON", System.Globalization.CultureInfo.InvariantCulture))).GetHicon(); ; }
        }

        public string Name
        {
            get
            {
                return "AppendOrRemoveCRLF";
            }
        }

        public string Version
        {
            get
            {
                return "1.0.0.0";
            }
        }

        public IBaseMessage Execute(IPipelineContext pContext, IBaseMessage pInMsg)
        {
            //Retrieve the message (stream) as a string.
            string messageBody = StreamHelper.MessageToString(pInMsg);
            string resultMessageBody = messageBody;
            //Do the stuff!!!
            if (Append)
            {
                System.Diagnostics.EventLog.WriteEntry("PipeComp", "Append");
                resultMessageBody = CRLFHelper.AppendCRLF(messageBody);
            }
            else
            {
                System.Diagnostics.EventLog.WriteEntry("PipeComp", "Remove");
                resultMessageBody = CRLFHelper.RemoveLastCRLF(messageBody);
            }
            StreamHelper.StringToMessage(resultMessageBody, ref pInMsg);
            return pInMsg;
        }

        public void GetClassID(out Guid classID)
        {
            classID = new Guid("8dc0ddd4-56ca-443f-af56-f0a28d20301f");
        }

        public void InitNew()
        {
            
        }

        public void Load(IPropertyBag propertyBag, int errorLog)
        {
            //
            try
            {
                object val = null;
                propertyBag.Read("Append", out val, 0);
                if (val != null)
                    Append = (bool)val;
            }
            catch (ArgumentException)
            {

                return;
            }



        }

        public void Save(IPropertyBag propertyBag, bool clearDirty, bool saveAllProperties)
        {
            //
            propertyBag.Write("Append", (object)Append);
        }

        public IEnumerator Validate(object projectSystem)
        {
            throw new NotImplementedException();
        }
    }
}



```


```csharp

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Kursus.PipelineComponents.Helpers
{
    public class CRLFHelper
    {
        public static string RemoveLastCRLF(string input)
        {
            input = input.TrimEnd(new char[] { ' ', '\r', '\n' });
            return input;
        }

        public static string AppendCRLF(string input)
        {
            input = RemoveLastCRLF(input);
            input += "\r\n";
            return input;
        }
    }
}


```


```csharp

using Microsoft.BizTalk.Message.Interop;
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Kursus.PipelineComponents.Helpers
{
    public class StreamHelper
    {
        public static  string MessageToString(IBaseMessage pInMsg)
        {

            IBaseMessagePart bodyPart = pInMsg.BodyPart;
            if (bodyPart != null)
            {
                Stream originalStrm;
                originalStrm = null;
                originalStrm = pInMsg.BodyPart.GetOriginalDataStream();
                System.IO.StreamReader reader = new StreamReader(originalStrm, System.Text.Encoding.UTF8);

                return reader.ReadToEnd();
            }
            else
            {
                return "";
            }
        }

        public static void StringToMessage(string s, ref IBaseMessage msg)
        {
            MemoryStream msOut = new MemoryStream();
            byte[] bts = System.Text.Encoding.UTF8.GetBytes(s);
            msOut.Seek(0, SeekOrigin.Begin);
            msOut.Write(bts, 0, bts.Length);
            msOut.Position = 0;
            msg.BodyPart.Data = msOut;
            long i = msg.BodyPart.Data.Length; //Crazy hack
        }
    }
}


```

[Back to top](#table-of-content)

## Services

### Exposing a Schema as a SOAP WebService

1. Open *BizTalk WCF Service Publishing Wizard*
2. Choose BasicHttp for SOAP1.0 (WebHttp for REST)
3. Enable on-premise.....
4. Choose *Publish Schema as WCF Service*
5. Delete *Operation1* if one-way is needed and add new one-way method
6. Right-Click *Request* and select schema type
7. Find the compiled .dll containing the schema and choose the appropriate schema
8. Find an appropriate namespace or leave it
9. Allow anonymous
10. Click Finish

11. In IIS refresh *Default Web Site*
12. Browse the created .svc file
13. Create an **Application Pool* with credentials that are member of the *Windows Group* specified under the **Isolated Host**
14. Use the newly created Application Pool with your Application
15. Create the appropriate *receive location* and enable it
16. Once you are able to browse your service you're finished!



### Consuming a Web Service

1. Create a new BizTalk project 
2. Add>Add generated Items>Consume WCF Service
3. Use the following URL: http://xemmel.com/webservices/postnumber.asmx 
4. Click Get
5. Schema should now be generated.
6. Create an instance of GetPostNumber (set root reference). Save the sample as an .XML file
7. Create new Receive Port (A) with a FILE receive location
8. Create a **Solicit-Response** Send Port (B) using the WCF-BasicHttp Adapter. Set the address as above and find the *SOAP Action Header* in the generated binding file (Subscribe to: BTS.ReceivePortName = A)
9. Create a new FILE Send Port (C) subscribe to BTS.SPName = B
10. Start everything and submit your sample with a city!! test that it works!!




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

### BizTalk message type

Namespace#Root element

example

> http://dti.dk/schemas/v10#Order



```powershell 
 Get-Service -Name "BTS*" | Out-GridView -PassThru | restart-service
 ``` 
 
 [Back to top](#table-of-content)
 
 ## Final test
 
 
 ```xml
 
 <ns0:Order xmlns:ns0="http://finaldemo">
	<ID>100</ID>
	<Customer>McD</Customer>
	<Type>Normal|Special</Type>
	<Line no="1">
		<ItemNo>10r40</ItemNo>
		<Qty>40.5</Qty>
	</Line>
	<Line no="2">
		<ItemNo>10rt</ItemNo>
		<Qty>400</Qty>
		<Desc>Not important</Desc>
	</Line>
</ns0:Order>
 
 ```
 
 1. Create an instance of the document above
 2. Create a matching schema make sure that the document validates
 3. Promote Type
 4. Make a flow (1 Receive Port, 2 Send Ports. SP1 subscribes to everything from the Receive Port. SP2 subscribes ONLY to Type = Special)
 5. Test and place your name tag on your screen
 
 
 6. Create a Flat file schema that matches the following sample.
 
 ```
 100;McD;Normal
 OL:1;10r40;40.5
 OL:2;10rt;400
 
 
 ```
 
 7. Test and deploy the Schema.
 8. Create a new Receive Pipeline, use the FFDisassembler and choose the FFSchema just created.
 9. Deploy the Pipeline
 10. Create a new Receive Location, *.txt use the newly created Pipeline and test that you get XML out in the Send Port when submitting the FF document
 
 
 11. Create a new map (FFOrder_to_IOrder) map from the FFSchema to the first created schema.
 12. Deploy the map and apply it to the Receive Port
 13. Test that a FF with Type = Special is sent to SP2
 14. Think about when the Type is promoted in this situation?
 
 15. Create an envelope-schema (remember you only need the root element and nothing else) for the following:
 
 ```xml
 
 <ns1:Batch xmlns:ns1="http://finalDemo/Batch">
  <ns0:Order xmlns:ns0="http://finaldemo">
	<ID>100</ID>
	<Customer>McD</Customer>
	<Type>Normal|Special</Type>
	<Line no="1">
		<ItemNo>10r40</ItemNo>
		<Qty>40.5</Qty>
	</Line>
	<Line no="2">
		<ItemNo>10rt</ItemNo>
		<Qty>400</Qty>
		<Desc>Not important</Desc>
	</Line>
</ns0:Order>
 <ns0:Order xmlns:ns0="http://finaldemo">
	<ID>100</ID>
	<Customer>McD</Customer>
	<Type>Normal|Special</Type>
	<Line no="1">
		<ItemNo>10r40</ItemNo>
		<Qty>40.5</Qty>
	</Line>
	<Line no="2">
		<ItemNo>10rt</ItemNo>
		<Qty>400</Qty>
		<Desc>Not important</Desc>
	</Line>
</ns0:Order>
 </ns1:Batch>
 
 ```
 
 16. Deploy the Schema and test that you can debatch the document
 
 [Back to top](#table-of-content)