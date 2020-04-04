# oracle-plsql-call-external-url
oracle-plsql-call-external-url

Note: 
-using oracle 11gr2

-schema: TST

-all command on oracle sqlplus 


1. Grant ACL
```
sqlplus /nolog
conn /as sysdba
grant execute on utl_http to tst;
grant execute on dbms_lock to tst;
exit
```

2. Create ACL
```
conn /as sysdba
BEGIN
  DBMS_NETWORK_ACL_ADMIN.create_acl (
    acl          => 'dedi_utl_http.xml', 
    description  => 'ACL to www.example.com',
    principal    => 'TST',  --user schema
    is_grant     => TRUE, 
    privilege    => 'connect',
    start_date   => SYSTIMESTAMP,
    end_date     => NULL);
end;
/
begin
  DBMS_NETWORK_ACL_ADMIN.assign_acl (
    acl         => 'dedi_utl_http.xml',
    host        => 'www.example.com', 
    lower_port  => 80,
    upper_port  => NULL);    
end; 
/
```

3. Create Procedure and Function
```
sqlplus /nolog
conn tst

create or replace procedure dedi_cek_data
( vData in varchar2
--,vUmur number
) is
  req utl_http.req;
  res utl_http.resp;
  url varchar2(4000) := 'http://www.example.com/data/'||vData; 
  name varchar2(4000);
  buffer varchar2(4000); 
  --content varchar2(4000) := '{"nama":"'||vNama||'", "umur":"'||vUmur||'"}';
begin
  req := utl_http.begin_request(url, 'GET',' HTTP/1.1');
  utl_http.set_header(req, 'user-agent', 'mozilla/4.0'); 
  --utl_http.set_header(req, 'content-type', 'application/json'); 
  --utl_http.set_header(req, 'Content-Length', length(content));
 
  --utl_http.write_text(req, content);
  utl_http.write_text(req, '');
  res := utl_http.get_response(req);
  -- process the response from the HTTP call
  begin
    loop
      utl_http.read_line(res, buffer);
      dbms_output.put_line(buffer);
    end loop;
    utl_http.end_response(res);
  exception
    when utl_http.end_of_body 
    then
      utl_http.end_response(res);
  end;
end dedi_cek_data;
/

create or replace function dedi_cek_data2
( vData in varchar2
--,vUmur number
) return varchar2
is
  req utl_http.req;
  res utl_http.resp;
  url varchar2(4000) := 'http://www.example.com/data/'||vData; 
  name varchar2(4000);
  buffer varchar2(4000); 
  --content varchar2(4000) := '{"nama":"'||vNama||'", "umur":"'||vUmur||'"}';
begin
  req := utl_http.begin_request(url, 'GET',' HTTP/1.1');
  utl_http.set_header(req, 'user-agent', 'mozilla/4.0'); 
  --utl_http.set_header(req, 'content-type', 'application/json'); 
  --utl_http.set_header(req, 'Content-Length', length(content));
 
  --utl_http.write_text(req, content);
  utl_http.write_text(req, '');
  res := utl_http.get_response(req);
  -- process the response from the HTTP call
  begin
    loop
      utl_http.read_line(res, buffer);
      --dbms_output.put_line(buffer);
      return buffer;
    end loop;
    utl_http.end_response(res);
  exception
    when utl_http.end_of_body 
    then
      utl_http.end_response(res);
  end;
end dedi_cek_data2;
```

4. Test data

--test1
```
SQL> select dedi_cek_data2('sample') from dual;
```

result:
```
{"data":"data sample"}
```

--test2
```
SQL> set serverout on
SQL> exec dedi_cek_data('sample`);
```

result:
```
{"data":"data sample"}
```

Done.
