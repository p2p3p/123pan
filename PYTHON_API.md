# 123云盘 Python API 使用文档

本文档适用于 `pan123_core.py` 核心模块，**无需进入交互式 CLI**，直接在 Python 脚本中调用云盘所有功能。

## 快速开始

```python
from pan123_core import Pan123Core

# 方式一：直接传入账号密码（推荐）
core = Pan123Core(user_name="你的手机号", password="你的密码")
core.login()  # 登录成功后自动保存配置到 pan123_config.json

# 方式二：从已有 token 直接初始化
core = Pan123Core(authorization="Bearer eyJxxx")
core.init_login_state()

# 方式三：从配置文件加载
import json
with open("pan123_config.json") as f:
    cfg = json.load(f)
core = Pan123Core(
    user_name=cfg["userName"],
    password=cfg["passWord"],
    authorization=cfg.get("authorization", ""),
    protocol=cfg.get("protocol", "android"),
)
core.login()
```

## 配置文件格式

```json
{
  "userName": "",
  "passWord": "",
  "authorization": "Bearer eyJxxx",
  "deviceType": "22021211RG",
  "osVersion": "Android_12",
  "protocol": "android"
}
```

支持 `protocol` 切换：`"android"`（安卓协议，流量不限速）或 `"web"`（网页协议）。

## 初始化参数

```python
Pan123Core(
    user_name="",        # 用户名（手机号）
    password="",         # 密码
    authorization="",    # Bearer Token（有 token 可免登录）
    protocol="android",  # android / web
    device_type="",      # 设备型号（留空自动随机）
    os_version="",       # 安卓版本（留空自动随机）
)
```

## 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `file_list` | list | 当前目录文件列表（通过 `refresh()` 或 `load_more()` 填充） |
| `cwd_path` | str | 当前工作目录路径（只读属性，不是方法） |
| `cwd_id` | int | 当前目录 ID |
| `headers` | dict | 当前请求头 |

## 方法列表

### 登录/登出

| 方法 | 说明 |
|------|------|
| `login()` | 使用用户名密码登录，自动保存 token |
| `logout()` | 登出，清除 token |
| `check_login()` | 检查当前登录状态是否有效 |
| `init_login_state()` | 用已有 token 初始化登录状态（无需密码） |
| `clear_account()` | 清除账号信息 |
| `get_user_info()` | 获取当前用户信息（返回 Result，data 含 `Nickname`、`SpaceUsed` 等字段） |
| `get_current_config()` | 获取当前配置字典（直接返回 dict，不是 Result 格式） |

### 目录浏览

| 方法 | 说明 |
|------|------|
| `list_dir(parent_id, page, limit)` | 获取指定目录的单页文件列表。返回 `Result` → `data.items`（文件列表）, `data.total`（总数） |
| `list_dir_all(parent_id)` | 获取指定目录全部文件（自动翻页） |
| `refresh()` | 刷新当前目录，清空 `file_list` 并重新加载第一页 |
| `load_more()` | 加载下一页，追加到 `file_list` |
| `cd(index)` | 进入 `file_list` 中指定下标的文件夹（0-based），自动刷新列表 |
| `cd_root()` | 返回根目录 |
| `cd_up()` | 返回上级目录 |
| `get_folder_details(folder_id)` | 获取文件夹详情 |

### 文件操作

| 方法 | 说明 |
|------|------|
| `get_download_url(index)` | 获取 `file_list` 中指定文件的下载直链。返回 `Result` → `data.url` |
| `get_item_download_url(item)` | 获取单个文件/文件夹的下载直链 |
| `mkdir(name)` | 在当前目录创建子目录 |
| `trash_by_index(index)` | 按 `file_list` 下标删除文件（移入回收站） |
| `trash(file_data, delete=True)` | `delete=True` 移入回收站，`delete=False` 从回收站恢复。`file_data` 传文件条目字典（来自 `file_list` 或 `list_recycle()`） |
| `restore(file_id)` | 从回收站恢复文件 |
| `list_recycle()` | 获取回收站文件列表。返回 `Result` → `data` 为列表（直接就是数组） |
| `share(file_ids, share_pwd, expiration)` | 创建分享链接。返回 `Result` → `data.share_url` |
| `share_by_indices(indices, share_pwd)` | 按 `file_list` 下标列表创建分享 |
| `set_protocol(protocol)` | 切换 `"android"` / `"web"` 协议 |

### 上传

| 方法 | 说明 |
|------|------|
| `upload_file(file_path, duplicate, on_progress)` | 上传单个文件到当前目录 |
| `upload_directory(dir_path, duplicate, on_progress)` | 递归上传整个文件夹 |
| `upload_folder(dir_path, duplicate, on_progress)` | 同上，上传文件夹 |

参数说明：
- `duplicate`: `0`=自动重命名, `1`=跳过, `2`=覆盖
- `on_progress`: 进度回调函数

## 工具类 Pan123Tool

提供文件级下载功能，需传入 `Pan123Core` 实例：

```python
from pan123_core import Pan123Core, Pan123Tool

core = Pan123Core(user_name="手机号", password="密码")
core.login()
tool = Pan123Tool(core)
```

| 方法 | 说明 |
|------|------|
| `download_file(index, save_dir, ...)` | 下载 `file_list` 中指定文件到本地 |
| `download_directory(directory, save_dir, ...)` | 递归下载文件夹 |
| `download_item(item, save_dir, ...)` | 下载单个文件/文件夹 |
| `download_url(url, file_name, save_dir, ...)` | 从直链下载文件 |
| `load_config_from_file()` | 从配置文件加载 |
| `save_config_to_file()` | 保存配置到文件 |

下载参数：
- `save_dir`: 保存目录，默认 `"download"`
- `overwrite`: 是否覆盖已有文件，默认 `False`
- `skip_existing`: 是否跳过已有文件，默认 `False`
- `on_progress`: 进度回调

## 完整示例

### 列出文件

```python
from pan123_core import Pan123Core

core = Pan123Core(user_name="手机号", password="密码")
core.login()

# 刷新（填充 file_list）
core.refresh()

# 遍历当前目录（字段名全大写）
for item in core.file_list:
    ftype = "文件夹" if item["Type"] == 1 else "文件"
    size = item["Size"]
    print(f"[{ftype}] {item['FileName']} ({size} bytes)")
```

### 进入子目录并列出

```python
# 方式一：按 file_list 下标进入
core.refresh()
core.cd(3)  # 进入第4项（0-based），自动刷新列表

# 方式二：直接指定目录 ID
result = core.list_dir(parent_id=26604499)
for item in result["data"]["items"]:
    print(item["FileName"])
```

### 下载文件

```python
from pan123_core import Pan123Core, Pan123Tool

core = Pan123Core(user_name="手机号", password="密码")
core.login()
tool = Pan123Tool(core)

# 刷新列表
core.refresh()

# 下载第6个文件（0-based，下载到 ./download 目录）
tool.download_file(5, save_dir="./download")
```

### 上传文件

```python
core = Pan123Core(user_name="手机号", password="密码")
core.login()

# 上传单个文件到当前目录
core.upload_file("/path/to/file.txt")
```

### 创建分享

```python
core.refresh()
result = core.share_by_indices([2, 4], share_pwd="1234")
print(result["data"]["share_url"])
```

### 切换协议

```python
core.set_protocol("android")  # 安卓协议（流量不限速）
core.set_protocol("web")      # 网页协议
```

## 返回值格式

除 `get_current_config()` 外，所有方法统一返回 `Result` 字典：

```python
{
    "code": 0,       # 0=成功, <0=失败, >0=警告
    "message": "ok", # 结果描述
    "data": {...}    # 业务数据，失败时为 None
}
```

## 文件条目字段

列表中的每个文件/文件夹包含以下主要字段（**均为大写开头**）：

| 字段 | 类型 | 说明 |
|------|------|------|
| `FileId` | int | 文件唯一 ID |
| `FileName` | str | 文件/文件夹名称 |
| `Type` | int | 1=文件夹, 0=文件 |
| `Size` | int | 文件大小（字节） |
| `ParentFileId` | int | 父目录 ID |
| `CreateAt` | str | 创建时间 |
| `UpdateAt` | str | 更新时间 |
| `Etag` | str | 文件校验值 |

## 注意事项

1. 所有字段名均为**大写开头**（如 `FileId`、`FileName`、`Type`），注意区分大小写
2. `get_current_config()` 直接返回字典，不是 Result 格式
3. `list_recycle()` 返回的 `data` 是列表，不是 `{items: [...]}` 格式
4. 回收站彻底删除需调用 `/api/file/delete` 接口，服务端异步处理，不会立即从列表消失
5. 登录后请先调用 `refresh()` 或 `list_dir()` 填充 `file_list`，再使用 `cd()`、`get_download_url()` 等依赖下标的方法
6. `cwd_path` 是属性，不是方法，调用时不要加括号