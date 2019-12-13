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

### # function for authenticate
to verify whether authentic user or not via above hashing function
you gotta change "user table" to name of your user table,
"user name column" to name of your user name column,
and "password column" to name of your password column,
```
CREATE OR REPLACE FUNCTION AUTHENTICATE_USER
  (p_username in varchar2, 
   p_password in varchar2)
return boolean
is
  l_user_name       "user table"."user name column"%type    := upper(p_username);
  l_password        "user table"."password column"%type;
  l_hashed_password varchar2(1000);
  l_count           number;
begin
-- Returns from the AUTHENTICATE_USER function 
--    0    Normal, successful authentication
--    1    Unknown User Name
--    2    Account Locked
--    3    Account Expired
--    4    Incorrect Password
--    5    Password First Use
--    6    Maximum Login Attempts Exceeded
--    7    Unknown Internal Error
--
-- First, check to see if the user exists
    select count(*) 
      into l_count 
      from "user table"
      where "user name column" = l_user_name;
      
     if l_count > 0 then
          -- Hash the password provided
          l_hashed_password := hash_password(l_user_name, p_password);
 
          -- Get the stored password
          select "password column" 
            into l_password 
            from "user table" 
           where "user name column" = l_user_name;
  
          -- Compare the two, and if there is a match, return TRUE
          if l_hashed_password = l_password then
              -- Good result. 
              APEX_UTIL.SET_AUTHENTICATION_RESULT(0);
              return true;
          else
              -- The Passwords didn't match
              APEX_UTIL.SET_AUTHENTICATION_RESULT(4);
              return false;
          end if;
  
    else
          -- The username does not exist
          APEX_UTIL.SET_AUTHENTICATION_RESULT(1);
          return false;
    end if;
    -- If we get here then something weird happened. 
    APEX_UTIL.SET_AUTHENTICATION_RESULT(7);
    return false;
exception 
    when others then 
        -- We don't know what happened so log an unknown internal error
        APEX_UTIL.SET_AUTHENTICATION_RESULT(7);
        -- And save the SQL Error Message to the Auth Status.
        APEX_UTIL.SET_CUSTOM_AUTH_STATUS(sqlerrm);
        return false;
        
end authenticate_user;
```
