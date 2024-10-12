# TruTest_Node-Red
Node-Red flows to interact with Datamars TrueTest equipment <br>

I've captured communications between TruTest devices and TruTest Data Link and worked out how to retieve sessions. <br>

Some serial commands interrupted. Still a work in progress, may be wrong.
```
{ZA1} Turn on ack
{ZC1} Turn on CR LF 

{FPNA0} Session 1
{FPNA11} Session 12
{FPNA12} Session 13

{FPNR11} Session 12 number of records
{FPNR12} Session 13 number of records

{FF11} select session 12
{FF12} select session 13

{FIF1,F0,RD,RT} Select Columns
{FR0} Select row 1
{FN} print row / next
```
Connect to device and select Session (Session variable is an integer)
```
let session = msg.payload;
session --;
msg.payload = (
    "{ZA1}{ZC1}"+
    "{FH}{FH}{FH}{FH}{FH}" +
    "{SOCU}{SLFL0}{SLFS0}{SLFT0}" +
    "{SLFF0}{SLFI0}{SLFL1}{SLFS1}{SLFT1}{SLFF1}{SLFI1}" +
    "{FPNA" + session + "}" +
    "{FPNR" + session + "}" +
    "{FF" + session + "}" +
    "{FIF1,F0,RD,RT}" +
    "{FR0}"
);
return msg;
```
Send `{FR0}` to select first row (row0) <br>
Send `{FN}` to read selected row and select next <br>
Read Tags from serial port and save to context variable 
```
let data_save = flow.get("EID_Read") || [];
let data = msg.payload;

if (data != ["[]"]) {
    if (data.includes(",372") || data.includes(",969") || data.includes(",900")) {
        data = data.replace(/[\])}[{(]/g,"");
        data = data.split(',');
        //msg.payload = data;
    
        data_save[data[0]] = {
            "EID":data[1],
            "VID":data[2],
            "Date":data[3],
            "Time":data[4]
            };
        
        flow.set("EID_Read", data_save);
        
        msg.payload = data_save
        return msg;
    }

}
```
