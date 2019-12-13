# Business logic by SQL
when you develop an application by APEX

### # how to add sequence as key
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

### # function for hashing password
will use it for hashing password for creating, and comparing
```
CREATE OR REPLACE FUNCTION HASH_PASSWORD
  (p_user_name in varchar2,
   p_password  in varchar2)
return varchar2
is
  l_password varchar2(255);
  -- The following salt is an example. 
  -- Should probably be changed to another random string.
  l_salt  varchar2(255) := '2345USFGOJN2T3HW89EFGOBN23R5SDFGAKL';
begin
    --
    -- The following encryptes the password using a salt string and the 
    -- DBMS_OBFUSCATION_TOOLKIT. 
    -- This is a one-way encryption using MD5
    -- 
    l_password := utl_raw.cast_to_raw (
					dbms_obfuscation_toolkit.md5(
					  input_string => p_password ||
                                      substr(l_salt,4,14) ||
                                      p_user_name ||
                                      substr(l_salt,5,10)));
    return l_password;
end hash_password;
```
