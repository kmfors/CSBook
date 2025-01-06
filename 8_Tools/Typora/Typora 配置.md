

typora-plugin 用户自定义配置，在 `Typora/resources/plugin/global/setting/custom_plugin.user.toml` 进行添加



```yaml
[redirectLocalRootUrl]
enable = true
root = "D:\\1_resource\\csbooks"

[resourceOperation]
# 对于无后缀名的文件视为资源
collect_file_without_suffix = false
ignore_folders = [ ".git", "node_modules", ".obsidian" ]
```