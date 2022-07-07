# Tabs ด้านบน
> ### **Table :** `PCC_DOSS_CONTROL`
>> ถ้ามีค่าเป็น `'1'` Tabs จะเป็นสีแดง แล้วสามารถกดได้
> - `IS_BANKRUPT` พบบุคคลล้มละลาย
> - `IS_REVIVE` พบบุคคลฟิ้นฟู
> - `IS_MEDIATE` พบบุคคลไกล่เกลี่ย
> - `IS_CIVIL` พบบุคคลแพ่ง
> - `IS_FOUND_ASSET` ตรวจพบทรัพย์
> <p>&nbsp;</p>
***
<br />
<br />

# Lock สำนวน
> ### **Table :** `PCC_DOSS_CONTROL`
>> ถ้ามีค่าเป็น `'1'` จะไม่สามารถทำรายต่อไปได้
> - `DO_NOT_VASTED` ไม่สามารถทำรายการยึดได้
> - `DO_NOT_SALE` ไม่สามารถทำรายการจำหน่ายได้
> - `DO_NOT_ACCOUNTANCY` ไม่สามารถทำรายการบัญชีได้ได้
>
>> **งานยึด** 
>> <br />&emsp;&emsp;`ถ้าปลดล็อคทรัพย์แล้ว ไม่สามารถยืนยันยึดได้ `
>> <br />&emsp;&emsp;`ต้องเปลี่ยน DO_NOT_VASTED ให้เท่ากับ NULL ก่อน`
>
>> **งานจำหน่าย** 
>> <br />&emsp;&emsp;`ถ้าปลดล็อคทรัพย์แล้ว ไม่สามารถขายได้ `
>> <br />&emsp;&emsp;`ต้องเปลี่ยน DO_NOT_SALE ให้เท่ากับ NULL ก่อน`
>
>> **งานบัญชี** 
>> <br />&emsp;&emsp;`ถ้าปลดล็อคทรัพย์แล้ว ไม่สามารถเริ่มปฏิบัติงาน หรือบันทึกส่งตรวจงานได้ `
>> <br />&emsp;&emsp;`ต้องเปลี่ยน DO_NOT_ACCOUNTANCY ให้เท่ากับ NULL ก่อน`
> <p>&nbsp;</p>
***
<br />
<br />

# Lock ทรัพย์
## ยึด
> ### **จะตรวจสอบสถานะ Lock ที่ Table `CFC_CAPTION` เป็นหลัก**
>> **Column:** `ASSET_DEBT_FLAG` ***สถานะทรัพย์ลูกหนี้***
>> - `'1'` = ล็อคสถานะทรัพย์ 
>> - `'2'` = ปลดล็อคสถานะทรัพย์ 
>> - `'3'` = อยู่ระหว่างสอบถามคำสั่ง จพท. <br />
>
>> **Column:** `ASSET_LOCK_DATE` ***วันที่ล็อคสถานะทรัพย์*** <br />
>
>> **Column:** `ASSET_RELEASE_DATE` ***วันที่ปลดล็อคสถานะทรัพย์*** 
> <p>&nbsp;</p>
<br />

## จำหน่าย, บัญชี 
### **บัญชี** `ถ้ามีทรัพย์ที่ติดล็อคอยู่จะไม่สามารถบันทึกเริ่มปฏิบัติงาน และบันทึกส่งตรวจงานได้`
> ### **จะตรวจสอบสถานะ Lock ที่ Table `SHR_PROPERTY_CASE` เป็นหลัก**
>> **Column:** `ASSET_DEBT_FLAG` ***สถานะทรัพย์ลูกหนี้***
>> - `'1'` = ล็อคสถานะทรัพย์ 
>> - `'2'` = ปลดล็อคสถานะทรัพย์ 
>> - `'3'` = อยู่ระหว่างสอบถามคำสั่ง จพท. <br />
>
>> **Column:** `ASSET_LOCK_DATE` ***วันที่ล็อคสถานะทรัพย์*** <br />
>
>> **Column:** `ASSET_RELEASE_DATE` ***วันที่ปลดล็อคสถานะทรัพย์*** 
> <p>&nbsp;</p>
***
<br />
<br />

# Script
> ## **ปลด Tabs โดยใช้** `PCC_CASE_GEN`
>```SQL
>UPDATE PCC_DOSS_CONTROL pdc SET IS_BANKRUPT = NULL
>WHERE PCC_DOSS_CONTROL_GEN IN 
>(
>  SELECT  PCC_DOSS_CONTROL_GEN 
>  FROM  SHR_CASE sc 
>  WHERE  sc.PCC_CASE_GEN = :pccCaseGen  
>) 
>AND 0 = 
>(
>  SELECT  COUNT(*) 
>  FROM  SHR_CIVIL_PERSON_MAP scpm 
>  WHERE  scpm.IS_BANKRUPT = '1'
>  AND scpm.PCC_CASE_GEN = :pccCaseGen  
>);
> ```
***
<br />

> ## **ปลด Lock คนโดยใช้** `PCC_CASE_GEN && CARD_ID`
>```SQL
>UPDATE SHR_CIVIL_PERSON_MAP SET IS_BANKRUPT = NULL 
>WHERE PCC_CASE_GEN = :pccCaseGen
>AND SHR_CIVIL_PERSON_MAP_GEN IN 
>( 
>  SELECT SHR_CIVIL_PERSON_MAP_GEN 
>  FROM SHR_CIVIL_PERSON_MAP scpm 
>  LEFT JOIN  SHR_PERSON_MAP spm 
>  ON scpm.SHR_PERSON_MAP_GEN = spm.SHR_PERSON_MAP_GEN 
>  WHERE spm.CARD_ID = :cardId
>);
>```
***
<br />

> ## **ปลด Lock ทรัพย์`(ยึด)`โดยใช้** `CFC_CATION_GEN`
>```SQL
> UPDATE CFC_CAPTION 
> SET ASSET_DEBT_FLAG = '2', ASSET_RELEASE_DATE = SYSDATE 
> WHERE CFC_CAPTION_GEN = :cfcCaptionGen;
>```
***
<br />

> ## **ปลด Lock ทรัพย์`(จำหน่าย,บัญชี)`โดยใช้** `CFC_CATION_GEN`
>```SQL
> UPDATE SHR_PROPERTY_CASE 
> SET ASSET_DEBT_FLAG = '2', ASSET_RELEASE_DATE = SYSDATE 
> WHERE CFC_CAPTION_GEN = :cfcCaptionGen;
>```
***
<br />