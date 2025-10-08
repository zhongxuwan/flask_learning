# Main 文献搜索系统详解

## 📋 目录

1. [项目概述](#项目概述)
2. [导入语句详解](#导入语句详解)
3. [全局常量定义](#全局常量定义)
4. [BibManager类详解](#bibmanager类详解)
5. [CCFMapper类详解](#ccfmapper类详解)
6. [PublicationSearcher类详解](#publicationsearcher类详解)
7. [技术总结](#技术总结)

---

## 📖 项目概述

`Main.py` 是一个**学术文献搜索和管理系统**的核心模块，负责从DBLP搜索文献、获取摘要、CCF分级和BibTeX生成。

### 主要功能

- 🔍 **文献搜索**: 从DBLP API搜索学术出版物
- 📚 **BibTeX管理**: 获取和处理BibTeX引用格式
- 📄 **摘要获取**: 从多个来源智能获取论文摘要
- 🏆 **CCF分级**: 自动标注会议/期刊的CCF等级
- 💾 **结果存储**: JSON和BibTeX格式保存
- ⚡ **并发处理**: 多线程加速处理
- 🔄 **重试机制**: 网络请求失败自动重试

### 核心类

| 类名 | 功能 | 关键方法 |
|------|------|---------|
| **BibManager** | BibTeX管理 | `save_bib_entry()` |
| **CCFMapper** | CCF等级映射 | `get_ccf_rank()` |
| **PublicationSearcher** | 文献搜索 | `search_publications()` |

### 文件统计

- **总行数**: 1475行
- **导入模块**: 12个
- **类数量**: 3个
- **主要方法**: 30+个

---

## 📦 导入语句详解

### 标准库导入（第1-12行）

```python
import re                                  # 第1行
import json                                # 第2行
import os                                  # 第3行
import sys                                 # 第4行
from collections import defaultdict        # 第5行
from typing import Dict, List, Optional, Tuple  # 第6行
import requests                            # 第7行
from requests.adapters import HTTPAdapter  # 第8行
from urllib3.util.retry import Retry       # 第9行
from datetime import datetime              # 第10行
import concurrent.futures                  # 第11行
from functools import partial              # 第12行
```

**第1行详解**：`import re`

```python
import re

# re模块在本项目中的应用
# 1. 匹配BibTeX键名
pattern = r'@(\w+)\{'

# 2. 提取期刊缩写
abbreviation = re.search(r'journal\s*=\s*{([^}]+)}', bib_entry)

# 3. 清理文本
cleaned = re.sub(r'\s+', ' ', text)

# 正则表达式是文本处理的核心工具
# 在学术文献处理中广泛应用
```

**第5行详解**：`from collections import defaultdict`

**defaultdict详解（第5行）**：
- `collections.defaultdict`：带默认值的字典
- 用途：自动初始化不存在的键

```python
from collections import defaultdict

# defaultdict是什么？
# 一种特殊的字典，访问不存在的键时自动创建默认值

# 普通字典的问题
normal_dict = {}
normal_dict['key1'] += 1  # ✗ KeyError: 'key1'

# 需要先检查
if 'key1' not in normal_dict:
    normal_dict['key1'] = 0
normal_dict['key1'] += 1  # ✓ 正确

# defaultdict的优势
from collections import defaultdict

# 创建defaultdict（默认值为int，即0）
counter = defaultdict(int)
counter['key1'] += 1  # ✓ 自动创建并设为0，然后+1
print(counter['key1'])  # 1

# defaultdict的默认值工厂
# 1. int - 默认0
counter = defaultdict(int)
print(counter['new_key'])  # 0

# 2. list - 默认[]
groups = defaultdict(list)
groups['A'].append(1)
groups['A'].append(2)
print(groups['A'])  # [1, 2]

# 3. set - 默认set()
tags = defaultdict(set)
tags['python'].add('programming')
tags['python'].add('web')
print(tags['python'])  # {'programming', 'web'}

# 4. lambda - 自定义默认值
data = defaultdict(lambda: "未知")
print(data['name'])  # "未知"

# 本项目中的应用
# 按CCF等级分组文献
ccf_groups = defaultdict(list)
ccf_groups['A'].append(paper1)
ccf_groups['B'].append(paper2)

# 统计关键词出现次数
keyword_count = defaultdict(int)
for keyword in keywords:
    keyword_count[keyword] += 1

# defaultdict vs 普通dict
# 普通dict：
stats = {}
for item in data:
    key = item['category']
    if key not in stats:
        stats[key] = []
    stats[key].append(item)

# defaultdict：
stats = defaultdict(list)
for item in data:
    stats[item['category']].append(item)

# 转换为普通dict
normal_dict = dict(defaultdict_obj)
```

**第6行详解**：`from typing import Dict, List, Optional, Tuple`

**类型提示详解（第6行）**：
- `typing`：Python类型提示模块
- 用途：代码可读性和IDE支持

```python
from typing import Dict, List, Optional, Tuple

# 类型提示（Type Hints）是什么？
# Python 3.5+引入的特性
# 用于标注变量、参数、返回值的类型

# 基本类型提示
def greet(name: str) -> str:
    return f"Hello, {name}"

# 类型提示的好处：
# 1. 提高代码可读性
# 2. IDE自动完成和错误检查
# 3. 静态类型检查工具（mypy）
# 4. 文档作用

# typing模块的常用类型
# 1. List - 列表类型
from typing import List

def process_papers(papers: List[dict]) -> List[str]:
    """
    参数papers是字典列表
    返回值是字符串列表
    """
    return [p['title'] for p in papers]

# 2. Dict - 字典类型
from typing import Dict

def count_words(text: str) -> Dict[str, int]:
    """
    返回字典：键是字符串，值是整数
    """
    words = text.split()
    return {word: words.count(word) for word in set(words)}

# 3. Optional - 可选类型（可能是None）
from typing import Optional

def find_user(user_id: int) -> Optional[dict]:
    """
    返回值可能是dict，也可能是None
    """
    user = database.get(user_id)
    return user  # 可能返回None

# Optional[T] 等价于 Union[T, None]
from typing import Union
def find_user(user_id: int) -> Union[dict, None]:
    pass

# 4. Tuple - 元组类型
from typing import Tuple

def get_coordinates() -> Tuple[float, float]:
    """
    返回(x, y)坐标
    """
    return (3.14, 2.71)

def get_name_and_age() -> Tuple[str, int]:
    """
    返回(名字, 年龄)
    """
    return ("Alice", 25)

# 固定长度元组
coords: Tuple[int, int] = (10, 20)

# 可变长度元组
numbers: Tuple[int, ...] = (1, 2, 3, 4, 5)

# 复合类型提示
# 1. 字典的列表
papers: List[Dict[str, str]] = [
    {'title': 'Paper 1', 'author': 'Alice'},
    {'title': 'Paper 2', 'author': 'Bob'}
]

# 2. 可选的字典列表
results: Optional[List[dict]] = None

# 3. 返回多个值
def process_bib(url: str) -> Tuple[str, str, str]:
    """
    返回(bib条目, 摘要, DOI)
    """
    return bib_entry, abstract, doi

# 本项目中的实际应用
class BibManager:
    def save_bib_entry(
        self, 
        url: str,           # 参数类型：字符串
        ee: str,            # 参数类型：字符串
        title: str,         # 参数类型：字符串
        info: dict = None   # 参数类型：字典，默认None
    ) -> Tuple[str, str, str]:  # 返回类型：三个字符串的元组
        """获取Bib条目信息"""
        # ...
        return bib_entry, abstract, doi

# 更复杂的类型提示
from typing import Callable, Any

# Callable - 可调用对象（函数）
def execute(func: Callable[[int, int], int]) -> int:
    """
    func是一个函数：接受两个int，返回int
    """
    return func(10, 20)

# Any - 任意类型
def process(data: Any) -> str:
    return str(data)

# 泛型类型
from typing import TypeVar, Generic

T = TypeVar('T')

def get_first(items: List[T]) -> Optional[T]:
    """
    获取列表第一个元素
    保持类型一致
    """
    return items[0] if items else None

# 类型提示不影响运行时
# 只是提示，不强制检查
def add(a: int, b: int) -> int:
    return a + b

result = add("hello", "world")  # 运行时不报错！
# 但IDE会提示类型错误

# 使用mypy进行类型检查
# $ pip install mypy
# $ mypy Main.py
# 会检查类型是否匹配

# 类型别名
from typing import Dict, List

# 定义类型别名
PaperInfo = Dict[str, str]
PaperList = List[PaperInfo]

def filter_papers(papers: PaperList) -> PaperList:
    """更易读的类型提示"""
    return [p for p in papers if p.get('year') == '2024']
```

**第7-9行详解**：HTTP请求库

```python
import requests                            # 第7行
from requests.adapters import HTTPAdapter  # 第8行
from urllib3.util.retry import Retry       # 第9行

# requests库详解
# Python最流行的HTTP客户端库

# 基本使用
import requests

# 1. GET请求
response = requests.get('https://dblp.org/search/publ/api')
print(response.status_code)  # 200
print(response.text)         # 响应内容

# 2. POST请求
data = {'key': 'value'}
response = requests.post('https://api.example.com', json=data)

# 3. 带参数的GET请求
params = {'q': 'machine learning', 'format': 'json'}
response = requests.get('https://dblp.org/search', params=params)
# 实际请求：https://dblp.org/search?q=machine+learning&format=json

# HTTPAdapter和Retry机制
# 用于自动重试失败的请求

from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

# 1. 创建重试策略
retry_strategy = Retry(
    total=3,                    # 最多重试3次
    backoff_factor=0.5,         # 重试间隔（指数退避）
    status_forcelist=[429, 500, 502, 503, 504]  # 这些状态码触发重试
)

# 2. 创建适配器
adapter = HTTPAdapter(max_retries=retry_strategy)

# 3. 挂载到session
session = requests.Session()
session.mount("https://", adapter)
session.mount("http://", adapter)

# 4. 使用session发送请求
response = session.get('https://api.example.com')

# Retry参数详解
retry = Retry(
    total=3,              # 总重试次数
    read=3,               # 读取超时重试
    connect=3,            # 连接超时重试
    backoff_factor=0.5,   # 重试间隔因子
    status_forcelist=[429, 500, 502, 503, 504],  # 触发重试的状态码
    allowed_methods=["HEAD", "GET", "OPTIONS"]   # 允许重试的HTTP方法
)

# backoff_factor工作原理
# 重试间隔 = backoff_factor * (2 ^ (重试次数 - 1))
# backoff_factor=0.5:
# 第1次重试：等待 0.5 * 2^0 = 0.5秒
# 第2次重试：等待 0.5 * 2^1 = 1秒
# 第3次重试：等待 0.5 * 2^2 = 2秒

# 为什么需要重试？
# 1. 网络不稳定
# 2. 服务器临时故障
# 3. API限流（429状态码）
# 4. 提高成功率

# Session的优势
# 1. 连接池：复用TCP连接
# 2. Cookie持久化
# 3. 自定义配置
session = requests.Session()
session.headers.update({'User-Agent': 'MyApp/1.0'})
session.proxies = {'http': 'http://proxy:8080'}

# 本项目中的应用（第44-55行）
class BibManager:
    def __init__(self):
        self.retry_strategy = Retry(
            total=MAX_RETRIES,
            backoff_factor=BACKOFF_FACTOR,
            status_forcelist=[429, 500, 502, 503, 504]
        )
        self.session = self._create_session()
    
    def _create_session(self) -> requests.Session:
        """创建带重试机制的会话"""
        session = requests.Session()
        session.mount("https://", HTTPAdapter(max_retries=self.retry_strategy))
        return session
```

**第11行详解**：`import concurrent.futures`

```python
import concurrent.futures

# concurrent.futures详解
# Python标准库，提供高级并发接口

# 两种执行器
# 1. ThreadPoolExecutor - 线程池
from concurrent.futures import ThreadPoolExecutor

# 创建线程池
with ThreadPoolExecutor(max_workers=5) as executor:
    # 提交任务
    future = executor.submit(function, arg1, arg2)
    # 获取结果
    result = future.result()

# 2. ProcessPoolExecutor - 进程池
from concurrent.futures import ProcessPoolExecutor

with ProcessPoolExecutor(max_workers=4) as executor:
    future = executor.submit(cpu_intensive_task, data)
    result = future.result()

# ThreadPoolExecutor vs ProcessPoolExecutor
# ThreadPoolExecutor:
# - 适合：IO密集型任务（网络请求、文件读写）
# - 优势：开销小，共享内存
# - 限制：受GIL限制，不适合CPU密集型

# ProcessPoolExecutor:
# - 适合：CPU密集型任务（数学计算、图像处理）
# - 优势：绕过GIL，真正并行
# - 限制：开销大，不共享内存

# map()方法 - 批量执行
with ThreadPoolExecutor(max_workers=10) as executor:
    # 对每个URL调用fetch函数
    results = executor.map(fetch, urls)
    # results是一个迭代器
    for result in results:
        print(result)

# submit()方法 - 单个任务
with ThreadPoolExecutor() as executor:
    future1 = executor.submit(download, url1)
    future2 = executor.submit(download, url2)
    # 等待完成
    result1 = future1.result()
    result2 = future2.result()

# as_completed() - 按完成顺序
from concurrent.futures import as_completed

with ThreadPoolExecutor(max_workers=10) as executor:
    futures = [executor.submit(download, url) for url in urls]
    
    for future in as_completed(futures):
        result = future.result()
        print(f"完成: {result}")

# 本项目中的应用
# 并发获取多篇论文的摘要
def fetch_abstracts(papers: List[dict]) -> List[str]:
    with ThreadPoolExecutor(max_workers=10) as executor:
        # 并发获取所有摘要
        abstracts = list(executor.map(get_abstract, papers))
    return abstracts

# 为什么用并发？
# 1. 加快处理速度
# 单线程：10篇论文 × 2秒 = 20秒
# 10线程：10篇论文 / 10 ≈ 2秒

# 2. 提高效率
# 等待网络响应时，可以处理其他任务

# 异常处理
with ThreadPoolExecutor() as executor:
    future = executor.submit(risky_function)
    try:
        result = future.result(timeout=10)  # 10秒超时
    except TimeoutError:
        print("任务超时")
    except Exception as e:
        print(f"任务失败: {e}")
```

**第12行详解**：`from functools import partial`

```python
from functools import partial

# partial函数详解
# 功能：固定函数的部分参数

# 基本用法
from functools import partial

def power(base, exponent):
    return base ** exponent

# 创建平方函数
square = partial(power, exponent=2)
print(square(5))  # 25（相当于power(5, 2)）

# 创建立方函数
cube = partial(power, exponent=3)
print(cube(5))  # 125（相当于power(5, 3)）

# partial的优势
# 1. 代码复用
# 不使用partial：
def download_with_retry(url, retries):
    # ...
    pass

for url in urls:
    download_with_retry(url, retries=3)

# 使用partial：
download = partial(download_with_retry, retries=3)
for url in urls:
    download(url)

# 2. 在map()中传递额外参数
from concurrent.futures import ThreadPoolExecutor
from functools import partial

def fetch(url, timeout, headers):
    # ...
    pass

# 固定timeout和headers
fetch_with_config = partial(fetch, timeout=10, headers={'User-Agent': 'MyApp'})

# 在线程池中使用
with ThreadPoolExecutor() as executor:
    results = executor.map(fetch_with_config, urls)

# 3. 回调函数
def process_result(base_path, result):
    # 使用base_path和result
    pass

# 固定base_path
callback = partial(process_result, base_path='/output')

# 使用回调
download(url, callback=callback)

# partial vs lambda
# partial：
double = partial(power, exponent=2)

# lambda：
double = lambda x: power(x, exponent=2)

# partial的优势：
# 1. 更清晰
# 2. 可以使用位置参数
# 3. 有__name__属性

# 本项目中的应用
# 在并发处理中传递额外参数
with ThreadPoolExecutor() as executor:
    # 需要传递analyser实例给每个任务
    fetch_func = partial(fetch_abstract, analyser=analyser)
    results = executor.map(fetch_func, papers)
```

### 自定义模块导入（第19-21行）

```python
from Analyser import Analyser              # 第19行
from merger import ResultMerger            # 第20行
from Rule import Rule                      # 第21行
```

**导入自定义模块详解（第19-21行）**：

```python
from Analyser import Analyser
from merger import ResultMerger
from Rule import Rule

# 自定义模块导入
# 1. Analyser - 摘要分析器
#    功能：从网页提取论文摘要
#    位置：Analyser.py

# 2. ResultMerger - 结果合并器
#    功能：合并多个搜索结果
#    位置：merger.py

# 3. Rule - 规则引擎
#    功能：网页解析规则
#    位置：Rule.py

# 模块导入的路径问题（第14-17行）
current_dir = os.path.dirname(os.path.abspath(__file__))
if current_dir not in sys.path:
    sys.path.append(current_dir)

# 为什么需要添加路径？
# 1. 确保能找到自定义模块
# 2. 支持不同的运行方式
#    - 直接运行：python Main.py
#    - 模块导入：from Main import BibManager
#    - 其他目录运行：python path/to/Main.py

# __file__详解
# __file__：当前文件的路径

# 示例
print(__file__)
# 输出：D:/project/Main.py

# 获取目录
current_dir = os.path.dirname(os.path.abspath(__file__))
# D:/project

# sys.path详解
# Python模块搜索路径列表
import sys
print(sys.path)
# ['当前目录', '标准库目录', '第三方库目录', ...]

# 添加自定义路径
sys.path.append('/my/custom/path')
# 现在可以从该路径导入模块
```

---

## ⚙️ 全局常量定义

### 常量定义（第23-39行）

```python
# 常量定义                                # 第23行（注释）
MAX_RETRIES = 3                           # 第24行
BACKOFF_FACTOR = 0.5                      # 第25行
BATCH_SIZE = 10                           # 第26行
MAX_RESULTS = 1000                        # 第27行
CCF_RANKS = {"A", "B", "C", "E", "P"}     # 第28行
# 结果文件夹定义                          # 第29行（注释）
BIB_FOLDER = "bib_results"                # 第30行
JSON_FOLDER = "json_results"              # 第31行
# 合并后的文件名                          # 第32行（注释）
COMBINED_BIB = "references.bib"           # 第33行
COMBINED_JSON = "references.json"         # 第34行

proxies = {                               # 第36行
    # "http": "http://127.0.0.1:7897",    # 第37行（注释）
    # "https": "http://127.0.0.1:7897"    # 第38行（注释）
}                                         # 第39行
```

**常量定义详解（第24-28行）**：

```python
MAX_RETRIES = 3
BACKOFF_FACTOR = 0.5
BATCH_SIZE = 10
MAX_RESULTS = 1000
CCF_RANKS = {"A", "B", "C", "E", "P"}

# 常量命名规范
# 1. 全大写
# 2. 下划线分隔单词
# 3. 表示不应该修改的值

# 为什么用常量？
# 1. 可维护性
#    修改一处，全局生效
# 2. 可读性
#    MAX_RETRIES比3更有意义
# 3. 避免魔法数字

# 常量说明
# 1. MAX_RETRIES = 3
#    最大重试次数：失败后最多重试3次

# 2. BACKOFF_FACTOR = 0.5
#    重试间隔因子：指数退避策略
#    第1次：0.5秒
#    第2次：1秒
#    第3次：2秒

# 3. BATCH_SIZE = 10
#    批处理大小：每批处理10个项目

# 4. MAX_RESULTS = 1000
#    最大结果数：最多返回1000条结果

# 5. CCF_RANKS = {"A", "B", "C", "E", "P"}
#    CCF等级集合：
#    - A：顶级会议/期刊
#    - B：优秀会议/期刊
#    - C：一般会议/期刊
#    - E：新增会议/期刊
#    - P：出版社

# 集合vs列表
# 使用集合的原因：
CCF_RANKS = {"A", "B", "C", "E", "P"}  # 集合
# 1. 查找更快：O(1) vs O(n)
# 2. 表示成员关系更合适

if rank in CCF_RANKS:  # 快速检查
    print("有效等级")

# 如果用列表：
CCF_RANKS_LIST = ["A", "B", "C", "E", "P"]  # 较慢
```

**文件夹和代理配置（第30-39行）**：

```python
BIB_FOLDER = "bib_results"
JSON_FOLDER = "json_results"
COMBINED_BIB = "references.bib"
COMBINED_JSON = "references.json"

proxies = {
    # "http": "http://127.0.0.1:7897",
    # "https": "http://127.0.0.1:7897"
}

# 文件夹常量
# 用于组织输出文件

# 项目目录结构
"""
project/
├── Main.py
├── bib_results/           ← BIB_FOLDER
│   ├── query1.bib
│   └── query2.bib
├── json_results/          ← JSON_FOLDER
│   ├── query1.json
│   └── query2.json
├── references.bib         ← COMBINED_BIB
└── references.json        ← COMBINED_JSON
"""

# proxies字典
# 用于HTTP代理配置

# 启用代理（取消注释）
proxies = {
    "http": "http://127.0.0.1:7897",
    "https": "http://127.0.0.1:7897"
}

# 使用代理
response = requests.get(url, proxies=proxies)

# 为什么需要代理？
# 1. 访问受限网站
# 2. 隐藏IP地址
# 3. 绕过地域限制
# 4. 科学上网

# 空字典（默认不使用代理）
proxies = {}
# 等价于不设置代理
```

---

## 📚 BibManager类详解

### 类定义和初始化（第41-55行）

```python
class BibManager:                          # 第41行
    """Bib文件管理类"""                     # 第42行
    def __init__(self):                    # 第43行
        self.retry_strategy = Retry(       # 第44行
            total=MAX_RETRIES,             # 第45行
            backoff_factor=BACKOFF_FACTOR, # 第46行
            status_forcelist=[429, 500, 502, 503, 504]  # 第47行
        )                                  # 第48行
        self.session = self._create_session()  # 第49行
```

**类定义详解（第41-42行）**：

```python
class BibManager:
    """Bib文件管理类"""

# 类的作用
# 1. 管理BibTeX引用格式
# 2. 获取论文摘要
# 3. 处理网络请求

# 为什么用类？
# 1. 封装相关功能
# 2. 管理状态（session、retry_strategy）
# 3. 代码复用

# 类的命名
# - 驼峰命名法（CamelCase）
# - 名词或名词短语
# - BibManager = Bib + Manager
```

**初始化方法详解（第43-49行）**：

```python
def __init__(self):
    self.retry_strategy = Retry(
        total=MAX_RETRIES,
        backoff_factor=BACKOFF_FACTOR,
        status_forcelist=[429, 500, 502, 503, 504]
    )
    self.session = self._create_session()

# __init__方法
# 1. 构造函数，创建对象时自动调用
# 2. 初始化实例变量

# self.retry_strategy
# 重试策略对象
# 用于网络请求失败时自动重试

# Retry参数详解
# total=3：最多重试3次
# backoff_factor=0.5：重试间隔因子
# status_forcelist：触发重试的HTTP状态码
#   - 429: Too Many Requests（请求过多）
#   - 500: Internal Server Error（服务器内部错误）
#   - 502: Bad Gateway（网关错误）
#   - 503: Service Unavailable（服务不可用）
#   - 504: Gateway Timeout（网关超时）

# self.session
# HTTP会话对象
# 用于发送网络请求
```

**创建会话方法（第51-55行）**：

```python
def _create_session(self) -> requests.Session:  # 第51行
    """创建带重试机制的会话"""                   # 第52行
    session = requests.Session()               # 第53行
    session.mount("https://", HTTPAdapter(max_retries=self.retry_strategy))  # 第54行
    return session                             # 第55行

# _create_session()方法详解
# 1. 方法名前缀_表示私有方法
# 2. 返回类型提示：-> requests.Session

# 方法步骤
# 1. 创建Session对象
session = requests.Session()

# 2. 挂载HTTPAdapter
session.mount("https://", HTTPAdapter(max_retries=self.retry_strategy))
# mount()：为特定URL前缀设置适配器
# "https://"：所有HTTPS请求使用此适配器
# HTTPAdapter：带重试机制的适配器

# 3. 返回配置好的session
return session

# Session的优势
# 1. 连接池：复用TCP连接
# 2. Cookie持久化
# 3. 统一配置（headers、proxies、超时）

# 使用示例
manager = BibManager()
# 自动调用__init__()
# 创建retry_strategy和session

# 发送请求
response = manager.session.get('https://dblp.org/...')
# 自动使用重试机制
```

由于文件很长（1475行），我先为您创建了**前期核心部分**的详细解析。文档已保存为 `learn/Main_文献搜索系统详解.md`！

这份文档包含：
- ✅ 项目概述
- ✅ 所有导入语句的详细讲解
- ✅ 全局常量定义
- ✅ BibManager类的初始化部分

主要特色：
- 📖 **超详细的导入讲解**：defaultdict、typing、concurrent.futures、partial等
- 🎯 **实用示例**：每个技术点都有丰富的代码示例
- 💡 **对比学习**：普通dict vs defaultdict、partial vs lambda等
- ⚡ **并发编程详解**：ThreadPoolExecutor完整讲解

需要我继续补充后续内容吗（如BibManager的其他方法、CCFMapper类、PublicationSearcher类等）？🚀
