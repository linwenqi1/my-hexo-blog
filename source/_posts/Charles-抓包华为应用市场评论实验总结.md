---
title: Charles 抓包华为应用市场评论实验总结
date: 2026-03-15 20:53:58
tags:
  - PacketCapture
  - AndroidSecurity
  - Frida
  - CharlesProxy
  - SSLPinning
categories: 
  - Computer Science
  - SecurityResearch
---

## 一、实验目标

在 Windows 11 环境下，利用 MuMu 模拟器（Android 12）配置 Charles 代理，通过手动安装系统级证书及 Frida 动态插桩技术绕过 SSL Pinning，实现对华为应用市场（`com.huawei.appmarket`）加密流量的明文解析，并导出为标准的 HAR 格式数据

---

<!-- more -->



## 二、环境准备

| 项目       | 说明                                 |
| ---------- | ------------------------------------ |
| 操作系统   | Windows 11                           |
| 抓包工具   | Charles v4.6+                        |
| 模拟器     | MuMu 模拟器（Android 12, x64 架构）  |
| Frida 版本 | 17.8.2 (Local & Server 必须严格一致) |
| 核心工具   | OpenSSL, adb、Python 3.10+           |
| 目标应用   | 华为应用市场                         |

---

## 三、实验步骤与关键操作

### 1. Charles 基础配置

- **代理端口**：`Proxy` → `Proxy Settings`，确保 HTTP Proxy 勾选，端口保持 `8888`。
- **访问控制**：`Proxy` → `Access Control Settings` 确保`Prompt to allow unauthorized connections`勾选

### 2. 模拟器网络代理设置

- 获取电脑局域网 IP：`Help` → `Local IP Address`（例如 `192.168.1.100`）。
- 在模拟器中：`设置` → `网络和互联网` → `互联网` → 点击当前 Wi-Fi → 修改网络 → 高级选项 → 代理设为手动。
  - 代理主机名：电脑 IP
  - 代理端口：`8888`
- 保存后，Charles 弹出连接确认框，点击 **Allow**。
- 完成后能看到Charles 左侧栏目中`Encrpyted`中能成功抓取模拟器内部的加密流量

### 3. SSL 证书安装与信任

#### 电脑端证书安装

- Charles 菜单：`Help` → `SSL Proxying` → `Install Charles Root Certificate`。
- 导入向导中：
  - 第一步：选择 **安装证书**， 选择当前用户
  - 第二步：选择 **将所有的证书放入下列存储** → 浏览 → 选择 **受信任的根证书颁发机构** → 确定。
  - 完成导入，出现安全警告时点击“是”。
- Charles 菜单：`Proxy` → `SSL Proxying Settings` 添加 `*:*`
- `Proxy` → `Start SSL Proxy`
- 如果证书导入成功，用电脑端浏览器访问https://www.baidu.com不会出现警告

#### 模拟器端证书安装（系统证书）

由于 Android 12 强制要求系统证书才能信任 HTTPS 流量，需将 Charles 证书手动注入系统目录。

- 从 Charles 导出证书：`Help` → `Save Charles Root Certificate...`，保存为 `charles.pem`。

- 计算哈希：将导出的 `charles.pem` 通过 OpenSSL 计算哈希值。

  ```bash
  openssl x509 -subject_hash_old -in charles.pem
  ```

  记下第一行输出，例如 `0adc448f`。

- 将 `charles.pem` 重命名为 `0adc448f.0`。

- MuMu 设置：重启模拟器，在**设备设置**开启“Root 权限”，并将“磁盘共享”设置为“可写系统盘”。

- 通过 adb 推送到模拟器系统目录：

  ```bash
  adb -s 127.0.0.1:7555 root
  adb -s 127.0.0.1:7555 remount #可能不需要
  adb -s 127.0.0.1:7555 push 0adc448f.0 /system/etc/security/cacerts/
  adb -s 127.0.0.1:7555 shell chmod 644 /system/etc/security/cacerts/0adc448f.0
  ```

- 重启模拟器使证书生效，如果前面步骤正确，尝试在模拟器中访问其他普通的app, 能看到Charles 成功解密普通 App 的 HTTPS 流量。

### 4. 绕过 SSL Pinning

华为应用市场采用了 SSL Pinning，直接抓包 HTTPS 流量会被拒绝。使用frida 绕过。

#### 电脑端安装frida

```powershell
pip install frida frida-tools
```

将frida提示的路径加入环境变量PATH中

```powershell
frida --version
```

显示版本号代表成功

#### 启动 frida-server

- 访问 [frida 官方 Releases](https://github.com/frida/frida/releases)，下载与本地 frida 版本匹配的 `frida-server`，选择架构为 `x86_64`（对应 MuMu 模拟器）

- 下载对应模拟器架构（`x86_64`）的 frida-server，版本于客户端一致，推送到模拟器并运行：

  ```bash
  adb -s 127.0.0.1:7555 push frida-server /data/local/tmp/
  adb -s 127.0.0.1:7555 shell
  su
  chmod 755 /data/local/tmp/frida-server
  /data/local/tmp/frida-server &
  ```

- 保持该窗口运行。如果模拟器重启，需重新运行此脚本。

#### 使用frida禁用 SSL Pinning

- 电脑端使用 frida 综合脚本：

  ```bash
  frida --codeshare akabe1/frida-multiple-unpinning -f com.huawei.appmarket -U
  ```

### 5. 抓取评论数据

- 在模拟器中打开华为应用市场，进入目标应用详情页（如“抖音”），切换到“评论”标签。

- 手动向下滑动，加载多页评论。

- 在 Charles 中观察请求，找到评论接口（域名：`forum-api-drcn.jos.dbankcloud.com`）。

- 接口返回 JSON 格式数据，结构如下：

  ```json
  {
    "layoutData": [
      {
        "layoutName": "commentitemcard",
        "dataList": [
          {
            "commentUser": {
              "nickName": "一*******",
              "phone": "Mate 60 Pro", // 用户机型
              "operTimeStamp": "1773155264366" // 评论时间戳
            },
            "commentDetail": {
              "commentInfo": "1、擅自改规则...不消费就当流量打工仔。", // 评论正文
              "rating": "1", // 评分
              "versionName": "37.9.0", // App版本
              "approveCounts": 9 // 点赞数
            },
            "pubAddress": "贵州" // IP 归属地
          },
          {
            "commentUser": {
              "nickName": "吉*******",
              "phone": "Mate30-5G版"
            },
            "commentDetail": {
              "commentInfo": "整体是挺好，就是现在升级到最新的38.0.0版本，有特别明显的卡顿...",
              "rating": "2"
            },
            "pubAddress": "福建"
          }
        ]
      }
    ],
    "maxId": "6150884656745",
    "hasNextPage": 1 // 1代表还有下一页，可继续抓取
  }
  ```

### 6. 导出 HAR 文件

- 在 Charles 左侧 Structure 视图中，选中评论接口域名节点（或按住 Ctrl 多选所有评论请求）。
- 菜单：`File` → `Export Session` → 选择 **HTTP Archive (.har)** 格式 → 保存为 `comments.har`。

---

## 四、遇到的问题及解决方案

| 问题                                                   | 解决方案                                                     |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| `adb` 连接多个设备，命令报错                           | 使用 `-s 127.0.0.1:7555` 指定目标设备                        |
| 向 `/system` 推送证书时提示 `Read-only file system`    | 关闭 MuMu 模拟器，设置中开启“磁盘共享”为“可写系统盘”，重新启动 |
| Charles 电脑端安装证书时找不到“受信任的根证书颁发机构” | 将所有的证书放入下列存储后选择受信任的根证书颁发机构         |
| 华为应用市场加载评论时 Charles 中请求一直挂起或返回空  | 需要绕过 SSL Pinning，使用 frida 脚本                        |
| 应用市场评论接口返回数据被压缩/加密                    | 确保 Charles 已开启 SSL 解密（`Proxy` → `SSL Proxying Settings` 添加 `*:*`） |
| 评论接口只有 `{"rtnCode":0}` 等简单响应                | 抓到的可能是预检接口，需找到真正的评论数据接口               |

---

## 五、实验成果

1. **成功配置 Charles 抓包环境**，电脑端证书受信任，模拟器系统证书安装正确。
2. **绕过华为应用市场的 SSL Pinning**，能够正常抓取 HTTPS 流量。
3. **捕获到完整的评论数据接口**，返回 JSON 包含用户昵称、评分、评论内容、点赞数、回复等字段。
4. **导出 HAR 文件** `comments.har`，包含多页评论请求，可用于后续批量数据提取。

---

## 六、附录：关键命令速查

```bash
# 查看设备列表
adb devices

# 指定设备执行命令
adb -s 127.0.0.1:7555 shell

# 推送证书
adb -s 127.0.0.1:7555 push 0adc448f.0 /system/etc/security/cacerts/

# 启动 frida-server
adb -s 127.0.0.1:7555 shell
su
/data/local/tmp/frida-server &

# frida 通用绕过脚本
frida --codeshare akabe1/frida-multiple-unpinning -f com.huawei.appmarket -U
```

