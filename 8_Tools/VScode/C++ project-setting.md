


单文件头文件位置：

```json
{
    "clangd.fallbackFlags": [
    	"-I{includ-path}"
    ]
}
```



多头文件：

linux：需要配合cmake生成指定command文件，供clangd检索告知

msvc：这个无法配合cmake，只能是在setting里配置头文件


Too many errors emitted, stopping now [clang: fatal_too_many_errors]: 

```json
CompileFlags:
    Add: -ferror-limit=0
```
