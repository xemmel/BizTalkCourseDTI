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
    Order (Infix, Hex delimiter 0x0D 0x0A)
        OrderHeader (Infix, Char delimiter ;)
            OrderID
            Customer
        OrderLine (Infix, Hex delimiter 0x0D 0x0A  Tag Identifier: OL:)
            ItemNo
            Quantity

5. Validate your sample document with the schema (see that you get correct xml)
6. Create a new Receive Pipeline