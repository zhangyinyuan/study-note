# Linux历史记录

## 使用 HISTSIZE 和 HISTFILESIZE 将这两个环境变量设置为 0，这样将不会记录任何历史记录
> 以在你的 shell 配置文件（如 ~/.bashrc 或 ~/.bash_profile）中添加以下行
```
export HISTSIZE=0
export HISTFILESIZE=0
```

刷新终端
```
source ~/.bashrc
```

## 临时禁用历史记录
```
set +o history
```

## 临时启用历史记录
```
set -o history
```

## 删除历史文件
```
rm ~/.bash_history
```
