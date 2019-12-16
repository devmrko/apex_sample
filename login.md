# login logic by SQL
when you develop an application by APEX

### # how to authenticate user
  * create sequence for user table
```
create sequence "sequence name of user table"
```
  * create user table
```
CREATE TABLE  "USERS" 
( "USER_ID"   number, 
  "USER_NAME" varchar2(255) NOT NULL ENABLE, 
  "PASSWORD"  varchar2(255) NOT NULL ENABLE, 
  PRIMARY KEY ("USER_ID") USING INDEX  ENABLE, 
  CONSTRAINT "USERS_U1" UNIQUE ("USER_NAME") USING INDEX  ENABLE
)
```
  * create trigger for insert user
```
CREATE OR REPLACE EDITIONABLE TRIGGER  "trigger name" 
  before insert on users 
  for each row 
begin 
  -- Get a unique sequence value to use as the primary key for inserting key for new user
  select "user sequence name".nextval into :new.user_id from dual; 
  -- Make sure to save the username in upper case
  :new.user_name := upper(:new.user_name); 
  -- Hash the password so we are not saving clear text
  :new.password := hash_password(upper(:new.user_name), :new.password); 
end; 
```
  * enable above trigger
```
ALTER TRIGGER  "trigger name" ENABLE
```
  * create trigger for hashing updated password
```
CREATE OR REPLACE EDITIONABLE TRIGGER  "trigger name" 
  before update of password on isv_se
  for each row 
begin 
  -- Hash the password so we are not saving clear text
  :new.password := hash_password(upper(:old.username), :new.password); 
end;
```
  * insert user(when user data is inserted, above trigger will hash the password, and add new sequence as the pk)
```
Insert into USERS (USER_NAME,PASSWORD) values ('user name', 'password')
```
