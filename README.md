# Teknologisk BizTalk Course March 2018

## Table of content
1. [Restart BizTalk](#powershell-restart-biztalk)
5. [Promotion](#promotion)
6. [Pipeline](#pipeline)
7. [Flat Files](#flat-files)

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
3. set Envelope to true
4. Set Body X-Path to Batch
5. Deploy Schema
6. Make sure that the actual messages inside the Batch message is valid messages according to your schema(s)
7. Submit the batch and verify that several messages are send to your Send Port
8. Make one of the messages invalid (change the namespace)
9. Submit again
10. Verify that the whole batch is 