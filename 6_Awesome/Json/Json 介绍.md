```c++
const char* json = "{\"project\":\"rapidjson\",\"stars\":10}";
```

反斜杠是起到转义字符的作用。相当于

```json
{
    "project": "rapidjson",
    "start": 6
}
```

