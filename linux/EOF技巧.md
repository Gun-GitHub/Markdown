如果直接使用 `EOF` 而不加引号，Bash 会解析变量和命令替换，导致执行结果被写入文件。

如:

```sh
cat << 'EOF' > output_file.sh
echo "---> get jupyter path <---"
jupyter_path=$(whereis jupyter)
_jupyter=${jupyter_path:9:$((${#jupyter_path}))}
EOF
```

写入文件的结果是:

```sh
echo "---> get jupyter path <---"
jupyter_path=
_jupyter=${jupyter_path:9:$((${#jupyter_path}))}
```

为了避免这种情况，我们需要确保源码被原样写入文件，而不是执行结果。

### 正确的方式：使用 `'EOF'` 防止变量替换和命令执行

在 `EOF` 前加上单引号（`'EOF'`），可以确保 Bash 不会解析变量或执行命令，而是将内容原样写入文件。

```sh
cat << 'EOF' > output_file.sh
echo "---> get jupyter path <---"
jupyter_path=$(whereis jupyter)
_jupyter=${jupyter_path:9:$((${#jupyter_path}))}
EOF
```

### 解释：

1. **`'EOF'`**：单引号包裹的 `EOF` 告诉 Bash 不要解析其中的变量或命令，而是将内容原样写入文件。
2. **`> output_file.sh`**：将内容写入 `output_file.sh` 文件。

### 验证：

运行上述命令后，`output_file.sh` 文件的内容将是：

```sh
echo "---> get jupyter path <---"
jupyter_path=$(whereis jupyter)
_jupyter=${jupyter_path:9:$((${#jupyter_path}))}
```

这些内容会被原样写入文件，而不会被执行或解析。

### 关键点：

- 使用 `'EOF'` 而不是 `EOF`，确保源码被原样写入文件。
- 如果使用 `EOF`（不加单引号），Bash 会解析变量和命令替换，导致执行结果被写入文件。

希望这能解决你的问题！如果还有其他疑问，请随时告诉我。
