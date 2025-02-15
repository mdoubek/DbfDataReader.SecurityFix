# DbfDataReader.SecurityFix

[![NuGet](https://img.shields.io/nuget/dt/DbfDataReader.SecurityFix.svg)](https://www.nuget.org/packages/DbfDataReader.SecurityFix)  
[![NuGet](https://img.shields.io/nuget/vpre/DbfDataReader.SecurityFix.svg)](https://www.nuget.org/packages/DbfDataReader.SecurityFix)  

**DbfDataReader.SecurityFix** is a **security-patched version** of [DbfDataReader](https://github.com/yellowfeather/DbfDataReader).  
This version **fixes vulnerabilities** in `System.Text.RegularExpressions`, ensuring safer processing of DBF (dBase) database files.  

---

## ** Installation**
You can install this package via NuGet:

```powershell
dotnet add package DbfDataReader.SecurityFix --version 2.0.1


Usage, to get summary info:

```csharp
var dbfPath = "path\\file.dbf";
using (var dbfTable = new DbfTable(dbfPath, Encoding.UTF8))
{
    var header = dbfTable.Header;

    var versionDescription = header.VersionDescription;
    var hasMemo = dbfTable.Memo != null;
    var recordCount = header.RecordCount;

    foreach (var dbfColumn in dbfTable.Columns)
    {
        var name = dbfColumn.ColumnName;
        var columnType = dbfColumn.ColumnType;
        var length = dbfColumn.Length;
        var decimalCount = dbfColumn.DecimalCount;
    }
}
```

and to iterate over the rows:

```csharp
var skipDeleted = true;

var dbfPath = "path/file.dbf";
using (var dbfTable = new DbfTable(dbfPath, Encoding.UTF8))
{        
    var dbfRecord = new DbfRecord(dbfTable);

    while (dbfTable.Read(dbfRecord))
    {
        if (skipDeleted && dbfRecord.IsDeleted)
        {
            continue;
        }

        foreach (var dbfValue in dbfRecord.Values)
        {
            var stringValue = dbfValue.ToString();
            var obj = dbfValue.GetValue();
        }
    }
}
```

There is also an implementation of DbDataReader:

```csharp
var options = new DbfDataReaderOptions
{
    SkipDeletedRecords = true
    // Encoding = EncodingProvider.GetEncoding(1252);
};

var dbfPath = "path/file.dbf";
using (var dbfDataReader = new DbfDataReader(dbfPath, options))
{
    while (dbfDataReader.Read())
    {
        var valueCol1 = dbfDataReader.GetString(0);
        var valueCol2 = dbfDataReader.GetDecimal(1);
        var valueCol3 = dbfDataReader.GetDateTime(2);
        var valueCol4 = dbfDataReader.GetInt32(3);
    }
}
```

which also means you can bulk copy to MS SqlServer:

```csharp
var options = new DbfDataReaderOptions
{
    SkipDeletedRecords = true
    // Encoding = EncodingProvider.GetEncoding(1252);
};

var dbfPath = "path/file.dbf";
using (var dbfDataReader = new DbfDataReader(dbfPath, options))
{
    using (var bulkCopy = new SqlBulkCopy(connection))
    {
        bulkCopy.DestinationTableName = "DestinationTableName";

        try
        {
            bulkCopy.WriteToServer(dbfDataReader);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error importing: dbf file: '{dbfPath}', exception: {ex.Message}");
        }
    }
}
```

Used by 

- DbfBulkCopy

    Command line application to bulk copy from DBF files to MS SqlServer
    
    https://github.com/yellowfeather/DbfBulkCopy

- dbf

    Command line utility to display DBF info and contents
    
    https://github.com/yellowfeather/dbf
