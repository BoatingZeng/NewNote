## vscode的cpp插件设置MinGW

```json
{
    "name": "Win32",
    "intelliSenseMode": "clang-x64",
    "includePath": [
        "${workspaceRoot}/src",
        "${workspaceRoot}/thirdparty/boost/include",
        "C:/MinGW/lib/gcc/mingw32/6.3.0/include/c++",
        "C:/MinGW/lib/gcc/mingw32/6.3.0/include/c++/mingw32",
        "C:/MinGW/lib/gcc/mingw32/6.3.0/include/c++/backward",
        "C:/MinGW/lib/gcc/mingw32/6.3.0/include",
        "C:/MinGW/include",
        "C:/MinGW/lib/gcc/mingw32/6.3.0/include-fixed"
    ],
    "defines": [
        "_DEBUG",
        "UNICODE",
        "__GNUC__=6",
        "__cdecl=__attribute__((__cdecl__))"
    ],
    "browse": {
        "path": [
            "${workspaceRoot}/src",
            "${workspaceRoot}/thirdparty/boost/include",
            "C:/MinGW/lib/gcc/mingw32/6.3.0/include",
            "C:/MinGW/lib/gcc/mingw32/6.3.0/include-fixed",
            "C:/MinGW/include/*"
        ],
        "limitSymbolsToIncludedHeaders": true,
        "databaseFilename": ""
    }
}
```

## markdown

### 表格模板
```
|HTTP Status Code|error_msg |备注   |
|:--------------:|:--------:|:-----:|
|200|无||
|400|userId或missionCode参数为空||
|500|服务器内部错误||

|参数名 |说明   |可为空 |备注   |
|:-----:|:-----:|:-----:|:-----:|
|userId|用户id|否||
|missionCode|任务代号|否||
|missionStartDate|任务开始日期(格式YYYY-MM-DD)|否||
|isVIP|是不是VIP，只有为1的时候认为是vip|是||
```

|HTTP Status Code|error_msg |备注   |
|:--------------:|:--------:|:-----:|
|200|无||
|400|userId或missionCode参数为空||
|500|服务器内部错误||

|参数名 |说明   |可为空 |备注   |
|:-----:|:-----:|:-----:|:-----:|
|userId|用户id|否||
|missionCode|任务代号|否||
|missionStartDate|任务开始日期(格式YYYY-MM-DD)|否||
|isVIP|是不是VIP，只有为1的时候认为是vip|是||