---
title: "Test. Hello Yuri"
excerpt_separator: "<!--more-->"
categories:
  - Data Science
tags:
  - R
  - Tutorial
---
This is a a brief description
<!--more-->

## Heading 2

adfafsa aedfa 

### Heading 3
adfsdf aa


```Delphi
      if Assigned(LTable) and (LTable.QueryInterface(IAuditableTable, LAuditableTable) = S_OK) then
      begin

        LTableAuditSettings := GetTableAuditSettings(LAuditedTableID, True {ACreateIfNeeded});

        LAuditEvent := TableEventEnum(LADOQuery.Fields[1].AsInteger);
        LAuditedFields := LTableAuditSettings.GetAuditedFields(LAuditEvent);

        LAuditedFields.CommaText := LADOQuery.Fields[2].AsString;


        for i := LAuditedFields.count-1 downto 0 do
        begin
          LAuditedFields[i] := Trim(LAuditedFields[i]);
          if (LAuditedFields[i] = '') or not LAuditableTable.IsAuditableField(LAuditedFields[i]) then
          begin
            LAuditedFields.Delete(i)    //manipulates the underlying list direclty
          end
        end;
      end;
  ```
