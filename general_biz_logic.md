# Business logic by SQL, when you develop an application by APEX

### how to add sequence as key
  * create sequence
```
create sequence "sequence name"
```
  * add sequence as a value
```
CREATE OR REPLACE EDITIONABLE TRIGGER  "trigger name" 
  before insert on "table name"             
  for each row  
begin   
  if :NEW."column name" is null then 
    select "sequence name".nextval into :NEW."column name" from sys.dual; 
  end if; 
end;
```
  * enable above trigger
```
ALTER TRIGGER  "trigger name" ENABLE
```
