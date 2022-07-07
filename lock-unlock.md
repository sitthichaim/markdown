# Tabs ด้านบน
> **Table :** `PCC_DOSS_CONTROL`
>> ถ้า = `'1'` Tabs จะเป็นสีแดง แล้วสามารถกดได้
> - `IS_BANKRUPT` พบบุคคลล้มละลาย
> - `IS_REVIVE` พบบุคคลฟิ้นฟู
> - `IS_MEDIATE` พบบุคคลไกล่เกลี่ย
> - `IS_CIVIL` พบบุคคลแพ่ง
> - `IS_FOUND_ASSET` ตรวจพบทรัพย์
> <p>&nbsp;</p>

***
>**ปลด Tabs โดยใช้ ** `PCC_CASE`
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

# งานยึด

# งานจำหน่าย


# งานบัญชี


