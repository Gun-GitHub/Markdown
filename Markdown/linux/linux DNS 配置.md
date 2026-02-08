在 Linux 系统中，`/etc/resolv.conf` 文件用于配置系统的 DNS（域名系统）解析器。这个文件包含了一系列指令，用于指定 DNS 服务器、搜索域以及解析器的行为。以下是对 `/etc/resolv.conf` 文件中所有常见字段的详细解释：

## 1. nameserver

### 语法

```sh
nameserver <DNS服务器IP>
```

### 说明

- **作用**：指定用于 DNS 查询的 DNS 服务器的 IP 地址。
- **数量限制**：通常最多可以指定 3 个 `nameserver`。当第一个服务器不可用时，系统会依次尝试后续的服务器。
- **示例**：
  ```sh
  nameserver 8.8.8.8
  nameserver 8.8.4.4
  ```

## 2. search

### 语法

```sh
search <域名1> <域名2> ...
```

### 说明

- **作用**：定义一组搜索域。当你在命令行或应用程序中输入一个不带域名的主机名时，系统会自动将这些搜索域附加到主机名后进行解析。
- **限制**：最多可以指定 6 个搜索域，每个域名最长 256 字符。
- **示例**：
  ```sh
  search example.com sub.example.com
  ```

## 3. domain

### 语法

```sh
domain <域名>
```

### 说明

- **作用**：指定一个默认的搜索域，用于解析不带域名的主机名。`domain` 指令和 `search` 指令类似，但 `domain` 只能指定一个域名，且如果同时存在 `domain` 和 `search`，`search` 的优先级更高。
- **示例**：
  ```sh
  domain example.com
  ```

## 4. options

### 语法

```sh
options <选项1> <选项2> ...
```

### 说明

- **作用**：配置 DNS 解析器的行为，可以设置多个选项以优化解析过程。
- **常见选项**：
  - **timeout:n**：设置等待 DNS 响应的超时时间，单位为秒。默认是 5 秒。
    ```sh
    options timeout:2
    ```
  - **attempts:n**：设置重试次数。默认是 2 次。
    ```sh
    options attempts:3
    ```
  - **rotate**：启用轮询使用多个 DNS 服务器，而不是按照顺序依次使用。
    ```sh
    options rotate
    ```
  - **timeout** 和 **attempts** 结合使用，可以更灵活地控制 DNS 查询的行为。
  - **ndots:n**：设置名称中点（`.`）的数量，决定主机名被视为完全限定域名（FQDN）的阈值。默认是 1。
    ```sh
    options ndots:2
    ```
  - **debug**：启用调试模式，详细记录 DNS 查询过程。通常用于故障排除。
    ```sh
    options debug
    ```
- **示例**：
  ```sh
  options timeout:3 attempts:5 rotate
  ```

## 5. sortlist

### 语法

```
sortlist <网络地址1> <掩码1> [<网络地址2> <掩码2> ...]
```

### 说明

- **作用**：定义 DNS 响应中 IP 地址的排序优先级。用于优化多网卡或多网络环境下的 DNS 响应顺序。
- **示例**：
  ```sh
  sortlist 192.168.1.0/255.255.255.0
  sortlist 10.0.0.0/255.0.0.0
  ```

## 6. options 中的其他高级选项

除了上述常见选项外，`options` 还支持一些高级配置，用于特定场景下的 DNS 解析优化。例如：

- **inet6**：启用 IPv6 DNS 查询。
  ```sh
  options inet6
  ```
- **edns0**：启用扩展 DNS 选项，支持更大的 DNS 包。
  ```sh
  options edns0
  ```

## 7. 示例 `resolv.conf` 文件

以下是一个综合示例，展示了如何配置不同的字段：

```sh
# 指定 DNS 服务器
nameserver 8.8.8.8
nameserver 8.8.4.4

# 定义搜索域
search example.com sub.example.com

# 设置 DNS 解析器选项
options timeout:2 attempts:3 rotate ndots:2

# 定义排序列表
sortlist 192.168.1.0/255.255.255.0 10.0.0.0/255.0.0.0
```

## 8. 其他注意事项

- **自动生成**：某些情况下，`/etc/resolv.conf` 可能由网络管理工具（如 `NetworkManager`、`systemd-resolved` 或 DHCP 客户端）自动生成和管理。在这种情况下，手动编辑可能会被覆盖。可以通过配置这些工具来持久化自定义设置。
- **符号链接**：在使用 `systemd-resolved` 的系统中，`/etc/resolv.conf` 可能是指向 `/run/systemd/resolve/stub-resolv.conf` 或其他位置的符号链接。确保了解当前系统的 DNS 解析架构，以避免配置冲突。
- **权限**：编辑 `/etc/resolv.conf` 通常需要超级用户权限。例如，使用 `sudo`：
  ```sh
  sudo nano /etc/resolv.conf
  ```

通过了解和正确配置 `/etc/resolv.conf`，可以优化 Linux 系统的 DNS 解析性能，确保网络应用和服务能够高效、可靠地进行域名解析。

## 9. `nameserver` 字段可以配置多个 DNS 服务器。

在 `/etc/resolv.conf` 文件中，你最多可以指定 **三个** `nameserver`。当系统需要解析域名时，会按照顺序依次尝试这些 DNS 服务器。如果第一个服务器没有响应，系统会尝试使用下一个服务器，直到所有服务器都尝试完或查询成功。

### 示例

```sh
nameserver 8.8.8.8       # 第一个 DNS 服务器
nameserver 8.8.4.4       # 第二个 DNS 服务器
nameserver 1.1.1.1       # 第三个 DNS 服务器
```

### 注意事项

- **顺序优先**：系统总是优先尝试第一个 `nameserver`，只有当第一个服务器不可用时，才会尝试后续的服务器。
- **数量限制**：虽然可以配置多个 `nameserver`，但根据标准最多只能指定三个。超过三个的配置通常会被忽略。
- **性能考虑**：如果第一个 DNS 服务器响应很慢或不可用，可能会影响 DNS 查询的速度，因此配置多个可靠的 `nameserver` 是个好习惯。

通过配置多个 `nameserver`，你可以增加 DNS 解析的冗余性，确保在某个 DNS 服务器不可用时，系统仍然能够正常解析域名。

## 10. 记得加锁,不然开机重启之后 DNS 解析会恢复

加锁

```sh
chattr +i /etc/resolv.conf
```

解锁

```sh
chattr -i /etc/resolv.conf
```
