# Replace all text in SQL Server (Replace-All-Text)

```
DECLARE @SearchText NVARCHAR(MAX) = N'DRM_FTS Info';
DECLARE @ReplaceText NVARCHAR(MAX) = N'Content_Type Example';

DECLARE @TableName NVARCHAR(512), @ColumnName NVARCHAR(512), @Sql NVARCHAR(MAX);

DECLARE cur CURSOR FOR
SELECT 
    QUOTENAME(s.name) + '.' + QUOTENAME(t.name) AS TableName,
    QUOTENAME(c.name) AS ColumnName
FROM 
    sys.columns c
    JOIN sys.types y ON c.user_type_id = y.user_type_id
    JOIN sys.tables t ON c.object_id = t.object_id
    JOIN sys.schemas s ON t.schema_id = s.schema_id
WHERE 
    y.name IN ('varchar', 'nvarchar', 'text', 'ntext')
    AND c.is_computed = 0
    AND OBJECTPROPERTY(t.object_id, 'IsMSShipped') = 0;

OPEN cur;
FETCH NEXT FROM cur INTO @TableName, @ColumnName;

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @Sql = '
        UPDATE ' + @TableName + '
        SET ' + @ColumnName + ' = REPLACE(' + @ColumnName + ', @SearchText, @ReplaceText)
        WHERE ' + @ColumnName + ' LIKE ''%'' + @SearchText + ''%''';

    EXEC sp_executesql @Sql, 
        N'@SearchText NVARCHAR(MAX), @ReplaceText NVARCHAR(MAX)', 
        @SearchText = @SearchText, 
        @ReplaceText = @ReplaceText;

    FETCH NEXT FROM cur INTO @TableName, @ColumnName;
END

CLOSE cur;
DEALLOCATE cur;
```
