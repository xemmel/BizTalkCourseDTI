# Teknologisk BizTalk Course March 2018

##Table of content
1. [Restart BizTalk](#powershell-restart-biztalk)
5. [Promotion](#promotion)
6. [Pipeline](#pipeline)

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

[Back to top](#table-of content)

## Pipeline

- Create a new Project (VS) Kursus.Common.Pipelines (Remember to Sign and specify the correct BizTalk Application) (Kursus.Common)
- Create a new Receive Pipeline inside the Project (ReceiveXMLValidate)
- Use the _XMLDisassembler_ and the _XMLValidate_ Pipeline Components in the correct stages (Disassemble, Validate)
- Deploy
- Use the Pipeline (Remember to reference the common Application) in your existing *Receive Location*, make an validating error in your sample xml document (validation error, it still needs to be valid xml).
- Test that your new Pipeline validates and fails when an invalid document is submitted