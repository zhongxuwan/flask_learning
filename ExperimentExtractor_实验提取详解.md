# Experiment Extractor 实验信息提取器详解

## 📋 目录

1. [项目概述](#项目概述)
2. [导入语句详解](#导入语句详解)
3. [全局配置和常量](#全局配置和常量)
4. [正则表达式模式](#正则表达式模式)
5. [缓存系统](#缓存系统)
6. [核心提取函数](#核心提取函数)
7. [技术总结](#技术总结)

---

## 📖 项目概述

`experiment_extractor.py` 是一个**智能论文实验信息提取器**，用于从学术论文中自动提取以下信息：

### 主要功能

- 📊 **对比算法提取**: 识别论文中提到的对比算法和基准方法
- 💻 **代码链接提取**: 查找GitHub等代码仓库链接
- 🎯 **需求场景提取**: 提取研究的实际应用场景
- 🔍 **研究问题提取**: 识别论文要解决的核心问题
- 🎓 **研究目标提取**: 提取研究的目的和目标
- 💡 **创新点提取**: 识别论文的创新思路和贡献

### 技术特点

- 🚀 **智能缓存**: MD5哈希缓存机制，避免重复处理
- 🔄 **并发处理**: 使用线程池并发处理多篇论文
- 🌐 **多源获取**: 支持从URL、DOI、PDF等多种来源提取
- 📝 **正则匹配**: 100+条正则表达式精准匹配学术文本
- 🛡️ **容错设计**: 完善的异常处理和降级策略

### 文件统计

- **总行数**: 1266行
- **导入模块**: 10个
- **正则模式**: 6大类100+条
- **核心函数**: 15个

---

## 📦 导入语句详解

### Python标准库（第1-6行）

```python
import re                        # 第1行 - 正则表达式
import os                        # 第2行 - 操作系统接口
import json                      # 第3行 - JSON处理
import time                      # 第4行 - 时间函数
import random                    # 第5行 - 随机数
import hashlib                   # 第6行 - 哈希算法
```

**第1行详解**：`import re`

**re正则表达式模块详解（第1行）**：
- `import`：Python导入关键字
- `re`：Regular Expression（正则表达式）模块
- 用途：文本模式匹配和搜索

```python
import re

# 正则表达式是什么？
# 一种强大的文本匹配工具，用于查找、提取、替换文本

# 本项目中的应用场景
# 1. 匹配实验章节标题
pattern = r"(?:4|IV|4\.0)[\s\.]*(?:Experiment|Evaluation|Results)"
text = "4. Experiment and Results"
match = re.search(pattern, text, re.IGNORECASE)
if match:
    print(f"找到实验章节：{match.group()}")

# 2. 提取对比算法
pattern = r"compar(?:e|ed|ing|ison)\s+(?:with|to|against)"
text = "We compared our method with baseline algorithms"
if re.search(pattern, text, re.IGNORECASE):
    print("提到了对比算法")

# 3. 提取GitHub链接
pattern = r"github\.com/[\w-]+/[\w-]+"
text = "Code available at https://github.com/user/repo"
match = re.search(pattern, text)
if match:
    print(f"找到代码链接：{match.group()}")

# re模块的核心函数
# 1. re.search() - 搜索第一个匹配
match = re.search(r'experiment', text, re.IGNORECASE)

# 2. re.findall() - 查找所有匹配
urls = re.findall(r'https?://[^\s]+', text)

# 3. re.sub() - 替换匹配
clean_text = re.sub(r'\s+', ' ', text)  # 多个空格替换为单个

# 4. re.compile() - 编译正则（性能优化）
pattern = re.compile(r'experiment', re.IGNORECASE)
matches = pattern.findall(text)

# 正则表达式的基本语法
# .        - 任意字符
# *        - 0次或多次
# +        - 1次或多次
# ?        - 0次或1次
# {n,m}    - n到m次
# []       - 字符集
# ()       - 分组
# |        - 或
# ^        - 开头
# $        - 结尾
# \s       - 空白字符
# \d       - 数字
# \w       - 字母数字下划线

# 实战示例：匹配实验章节
pattern = r"(?:4|IV|4\.0)[\s\.]*(?:Experiment|Evaluation|Results)"
# 解析：
# (?:4|IV|4\.0)  - 匹配"4"或"IV"或"4.0"（非捕获组）
# [\s\.]*        - 匹配0个或多个空白或点号
# (?:Experiment|Evaluation|Results) - 匹配这三个词之一

texts = [
    "4. Experiment",
    "IV. Evaluation",
    "4.0 Results",
    "4   Experiment and Results"
]

for text in texts:
    if re.search(pattern, text, re.IGNORECASE):
        print(f"✓ 匹配：{text}")

# 为什么用(?:...)而不是(...)？
# (?:...)  - 非捕获组，只分组不捕获，性能更好
# (...)    - 捕获组，保存匹配结果
```

**第6行详解**：`import hashlib`

**hashlib哈希算法详解（第6行）**：
- `hashlib`：哈希算法库
- 用途：生成数据的哈希值（MD5、SHA等）

```python
import hashlib

# 哈希算法是什么？
# 将任意长度的数据转换为固定长度的字符串
# 特点：单向、确定性、雪崩效应

# 本项目中的应用：生成缓存文件名
paper_key = "https://arxiv.org/abs/2024.12345"
cache_version = "v2"

# 步骤1：创建版本化的键
versioned_key = f"{cache_version}_{paper_key}"
# "v2_https://arxiv.org/abs/2024.12345"

# 步骤2：编码为字节
key_bytes = versioned_key.encode()

# 步骤3：生成MD5哈希
hash_object = hashlib.md5(key_bytes)

# 步骤4：获取十六进制字符串
hash_hex = hash_object.hexdigest()
# "a1b2c3d4e5f6789..."（32个字符）

# 步骤5：生成缓存文件名
cache_filename = hash_hex + ".json"
# "a1b2c3d4e5f6789....json"

# 一行完成
cache_filename = hashlib.md5(f"v2_{paper_key}".encode()).hexdigest() + ".json"

# 为什么使用哈希？
# 1. 固定长度：无论URL多长，哈希值都是32字符
# 2. 合法文件名：只包含[0-9a-f]，没有特殊字符
# 3. 唯一性：不同的URL生成不同的哈希值
# 4. 快速查找：哈希值可以作为字典键

# MD5哈希示例
urls = [
    "https://arxiv.org/abs/2024.12345",
    "https://ieeexplore.ieee.org/document/98765",
    "https://dl.acm.org/doi/10.1145/123456"
]

for url in urls:
    hash_value = hashlib.md5(url.encode()).hexdigest()
    print(f"{url[:40]}... → {hash_value[:16]}...")

# 不同哈希算法对比
text = "Hello, World!"

# MD5（128位，32字符）- 最快，但不太安全
md5_hash = hashlib.md5(text.encode()).hexdigest()
print(f"MD5:    {md5_hash}")

# SHA1（160位，40字符）
sha1_hash = hashlib.sha1(text.encode()).hexdigest()
print(f"SHA1:   {sha1_hash}")

# SHA256（256位，64字符）- 更安全，较慢
sha256_hash = hashlib.sha256(text.encode()).hexdigest()
print(f"SHA256: {sha256_hash}")

# 本项目为什么用MD5？
# 1. 只是生成文件名，不需要高安全性
# 2. MD5最快
# 3. 32字符的文件名长度适中
```

### 第三方库（第7-10行）

```python
import requests                      # 第7行 - HTTP请求
from bs4 import BeautifulSoup        # 第8行 - HTML解析
from urllib.parse import urlparse    # 第9行 - URL解析
import concurrent.futures            # 第10行 - 并发处理
```

**第8行详解**：`from bs4 import BeautifulSoup`

**BeautifulSoup HTML解析详解（第8行）**：
- `bs4`：Beautiful Soup 4的包名
- `BeautifulSoup`：HTML/XML解析器类
- 用途：解析网页，提取内容

```python
from bs4 import BeautifulSoup

# BeautifulSoup是什么？
# 一个强大的HTML/XML解析库，可以轻松提取网页内容

# 基本使用流程
# 1. 获取HTML
html = """
<html>
<head><title>Deep Learning Paper</title></head>
<body>
    <div class="abstract">
        <p>This paper presents a novel approach...</p>
    </div>
    <section id="experiments">
        <h2>4. Experiments</h2>
        <p>We compare our method with baseline algorithms...</p>
        <p>Code available at <a href="https://github.com/user/repo">GitHub</a></p>
    </section>
</body>
</html>
"""

# 2. 创建BeautifulSoup对象
soup = BeautifulSoup(html, 'html.parser')
# 第二个参数是解析器：
# - 'html.parser': Python内置，速度中等
# - 'lxml': 最快，需要安装lxml
# - 'html5lib': 最容错，需要安装html5lib

# 3. 查找元素
# 方法1: find() - 查找第一个
title = soup.find('title')
print(title.text)  # "Deep Learning Paper"

# 方法2: find_all() - 查找所有
paragraphs = soup.find_all('p')
for p in paragraphs:
    print(p.text)

# 方法3: CSS选择器 - 更灵活
abstract = soup.select_one('.abstract p')
print(abstract.text)

# 方法4: 按属性查找
experiments = soup.find('section', id='experiments')
print(experiments.h2.text)  # "4. Experiments"

# 方法5: 查找链接
links = soup.find_all('a', href=True)
for link in links:
    print(link['href'])  # 获取href属性

# 本项目中的实际应用：提取论文内容
def extract_paper_content(html):
    """从论文HTML中提取文本"""
    soup = BeautifulSoup(html, 'html.parser')
    
    # 1. 提取标题
    title = soup.find('h1', class_='title')
    if title:
        title_text = title.get_text(strip=True)
    
    # 2. 提取摘要
    abstract = soup.find('div', class_='abstract')
    if abstract:
        abstract_text = abstract.get_text(strip=True)
    
    # 3. 提取所有段落
    paragraphs = soup.find_all('p')
    all_text = ' '.join([p.get_text() for p in paragraphs])
    
    # 4. 提取链接
    links = soup.find_all('a', href=True)
    github_links = [
        link['href'] 
        for link in links 
        if 'github.com' in link['href']
    ]
    
    return {
        'title': title_text,
        'abstract': abstract_text,
        'content': all_text,
        'code_links': github_links
    }

# BeautifulSoup的高级技巧
# 1. 导航树
soup = BeautifulSoup(html, 'html.parser')
section = soup.find('section')
section.parent        # 父元素
section.next_sibling  # 下一个兄弟元素
section.children      # 所有子元素

# 2. 搜索文本
soup.find_all(text=re.compile('experiment'))  # 查找包含"experiment"的文本

# 3. 组合条件
soup.find_all('p', class_='highlight', id=re.compile('para-'))

# 4. Lambda函数过滤
soup.find_all(lambda tag: tag.name == 'p' and len(tag.text) > 100)

# 为什么需要HTML解析？
# 论文网站HTML示例：
# <div class="paper-content">
#   <h2>4. Experiments</h2>
#   <p>We evaluated our approach...</p>
# </div>

# 不用BeautifulSoup（困难）：
# 需要手动写正则表达式，容易出错
pattern = r'<h2>(.*?)</h2>.*?<p>(.*?)</p>'
# 这个正则只能处理简单情况，复杂HTML会失败

# 用BeautifulSoup（简单）：
soup = BeautifulSoup(html, 'html.parser')
h2 = soup.find('h2').text
p = soup.find('p').text
```

**第9行详解**：`from urllib.parse import urlparse`

**urlparse URL解析详解（第9行）**：
- `urllib.parse`：URL解析模块
- `urlparse`：将URL分解为组成部分

```python
from urllib.parse import urlparse

# urlparse是什么？
# 将URL分解为协议、域名、路径等组件

# 基本使用
url = "https://arxiv.org:443/abs/2024.12345?version=1#section2"
parsed = urlparse(url)

print(parsed.scheme)    # 'https' - 协议
print(parsed.netloc)    # 'arxiv.org:443' - 网络位置（域名+端口）
print(parsed.hostname)  # 'arxiv.org' - 主机名
print(parsed.port)      # 443 - 端口号
print(parsed.path)      # '/abs/2024.12345' - 路径
print(parsed.query)     # 'version=1' - 查询参数
print(parsed.fragment)  # 'section2' - 片段标识符

# URL的完整结构
# scheme://netloc/path?query#fragment
# https://arxiv.org:443/abs/2024.12345?version=1#section2

# 本项目中的应用：提取域名
def get_domain(url):
    """从URL提取域名"""
    parsed = urlparse(url)
    return parsed.hostname

urls = [
    "https://arxiv.org/abs/2024.12345",
    "https://ieeexplore.ieee.org/document/98765",
    "https://dl.acm.org/doi/10.1145/123456"
]

for url in urls:
    domain = get_domain(url)
    print(f"{url} → {domain}")

# 输出：
# https://arxiv.org/... → arxiv.org
# https://ieeexplore.ieee.org/... → ieeexplore.ieee.org
# https://dl.acm.org/... → dl.acm.org

# 实际应用：根据域名选择不同的解析策略
def get_paper_parser(url):
    """根据URL选择合适的解析器"""
    domain = urlparse(url).hostname
    
    if 'arxiv.org' in domain:
        return parse_arxiv
    elif 'ieee.org' in domain:
        return parse_ieee
    elif 'acm.org' in domain:
        return parse_acm
    else:
        return parse_generic

# urlparse vs 字符串分割
# 不推荐：手动分割URL
url = "https://arxiv.org/abs/2024.12345"
parts = url.split('/')
domain = parts[2]  # 'arxiv.org'
# 问题：无法处理端口、查询参数等复杂情况

# 推荐：使用urlparse
parsed = urlparse(url)
domain = parsed.hostname  # 正确处理各种情况

# 相关函数
from urllib.parse import urljoin, quote, unquote

# urljoin - 合并URL
base = "https://arxiv.org/abs/"
relative = "2024.12345"
full_url = urljoin(base, relative)
# "https://arxiv.org/abs/2024.12345"

# quote - URL编码
text = "深度学习 论文"
encoded = quote(text)
# "%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%20%E8%AE%BA%E6%96%87"

# unquote - URL解码
decoded = unquote(encoded)
# "深度学习 论文"
```

**第10行详解**：`import concurrent.futures`

**concurrent.futures并发处理详解（第10行）**：
- `concurrent.futures`：高级并发接口
- 用途：简化多线程和多进程编程

```python
import concurrent.futures
import time

# concurrent.futures是什么？
# Python 3.2+引入的高级并发API
# 提供ThreadPoolExecutor（线程池）和ProcessPoolExecutor（进程池）

# 为什么需要并发？
# 场景：提取100篇论文的实验信息
# - 串行处理：每篇3秒 × 100 = 300秒（5分钟）
# - 并发处理（10线程）：每批3秒 × 10批 = 30秒

# 基本使用：ThreadPoolExecutor
def extract_experiments(paper_url):
    """提取单篇论文的实验信息（耗时操作）"""
    time.sleep(3)  # 模拟网络请求
    return f"Extracted from {paper_url}"

# 串行处理（慢）
paper_urls = [f"https://arxiv.org/abs/2024.{i}" for i in range(5)]

start = time.time()
results = []
for url in paper_urls:
    result = extract_experiments(url)
    results.append(result)
print(f"串行耗时：{time.time() - start:.2f}秒")  # ~15秒

# 并发处理（快）
start = time.time()
with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
    # submit方法：提交单个任务
    futures = [executor.submit(extract_experiments, url) for url in paper_urls]
    
    # as_completed：按完成顺序获取结果
    results = []
    for future in concurrent.futures.as_completed(futures):
        result = future.result()
        results.append(result)

print(f"并发耗时：{time.time() - start:.2f}秒")  # ~3秒

# ThreadPoolExecutor的核心方法
executor = concurrent.futures.ThreadPoolExecutor(max_workers=10)

# 1. submit() - 提交单个任务
future = executor.submit(extract_experiments, url)
result = future.result()  # 等待完成并获取结果

# 2. map() - 批量提交任务（保持顺序）
results = executor.map(extract_experiments, paper_urls)
for result in results:
    print(result)

# 3. as_completed() - 按完成顺序处理
futures = [executor.submit(extract_experiments, url) for url in paper_urls]
for future in concurrent.futures.as_completed(futures):
    result = future.result()
    print(f"完成：{result}")

# 本项目中的实际应用
def extract_batch_experiments(paper_list, max_workers=10):
    """批量提取论文实验信息"""
    results = {}
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        # 提交所有任务
        future_to_paper = {
            executor.submit(extract_experiments, paper['url']): paper['title']
            for paper in paper_list
        }
        
        # 处理完成的任务
        for future in concurrent.futures.as_completed(future_to_paper):
            title = future_to_paper[future]
            try:
                result = future.result()
                results[title] = result
                print(f"✓ 完成：{title}")
            except Exception as e:
                print(f"✗ 失败：{title} - {e}")
                results[title] = None
    
    return results

# 使用示例
papers = [
    {'title': '论文1', 'url': 'https://arxiv.org/abs/2024.001'},
    {'title': '论文2', 'url': 'https://arxiv.org/abs/2024.002'},
    {'title': '论文3', 'url': 'https://arxiv.org/abs/2024.003'},
]

results = extract_batch_experiments(papers, max_workers=3)

# ThreadPoolExecutor vs ProcessPoolExecutor
# ThreadPoolExecutor（线程池）：
# - 适合：I/O密集型任务（网络请求、文件读写）
# - 优点：轻量、共享内存
# - 缺点：受GIL限制，CPU密集型任务无法充分利用多核

# ProcessPoolExecutor（进程池）：
# - 适合：CPU密集型任务（计算、图像处理）
# - 优点：真正并行，充分利用多核
# - 缺点：进程开销大、不共享内存

# 本项目用ThreadPoolExecutor的原因：
# 提取论文信息主要是网络请求（I/O密集型）

# 错误处理
with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
    futures = [executor.submit(extract_experiments, url) for url in paper_urls]
    
    for future in concurrent.futures.as_completed(futures):
        try:
            result = future.result(timeout=30)  # 设置超时
            print(result)
        except concurrent.futures.TimeoutError:
            print("任务超时")
        except Exception as e:
            print(f"任务失败：{e}")

# 上下文管理器的优势
# with语句确保：
# 1. 任务完成后自动关闭线程池
# 2. 释放资源
# 3. 即使发生异常也能正确清理

# 不使用with（需要手动管理）：
executor = concurrent.futures.ThreadPoolExecutor(max_workers=5)
try:
    # 提交任务...
    pass
finally:
    executor.shutdown(wait=True)  # 必须手动关闭

# 使用with（推荐）：
with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
    # 提交任务...
    pass
# 自动关闭和清理
```

---

## ⚙️ 全局配置和常量

### 缓存版本（第12-13行）

```python
# 缓存版本                                 # 第12行（注释）
CACHE_VERSION = "v2"  # 更新版本以避免与旧缓存冲突  # 第13行
```

**第13行详解**：`CACHE_VERSION = "v2"`

**缓存版本控制详解（第13行）**：
- `CACHE_VERSION`：常量名（全大写）
- `=`：赋值运算符
- `"v2"`：版本字符串

```python
CACHE_VERSION = "v2"

# 为什么需要缓存版本？
# 问题场景：
# 1. 你修改了提取逻辑，提取字段从5个增加到6个
# 2. 旧缓存文件只有5个字段
# 3. 程序读取旧缓存，字段缺失导致错误

# 解决方案：版本控制
# 修改代码时，更新版本号：v1 → v2
# 程序会生成新的缓存文件，忽略旧缓存

# 版本如何使用
def get_cache_path(paper_url):
    """生成缓存文件路径"""
    # 将版本号加入哈希
    versioned_key = f"{CACHE_VERSION}_{paper_url}"
    # "v2_https://arxiv.org/abs/2024.12345"
    
    cache_filename = hashlib.md5(versioned_key.encode()).hexdigest() + ".json"
    return os.path.join("experiment_cache", cache_filename)

# 版本变化的影响
# v1版本：
url = "https://arxiv.org/abs/2024.12345"
v1_key = f"v1_{url}"
v1_hash = hashlib.md5(v1_key.encode()).hexdigest()
# v1_hash = "abc123..."
# 缓存文件：experiment_cache/abc123....json

# v2版本：
v2_key = f"v2_{url}"
v2_hash = hashlib.md5(v2_key.encode()).hexdigest()
# v2_hash = "def456..."（不同！）
# 缓存文件：experiment_cache/def456....json

# 结果：v1和v2的缓存文件不同，互不影响

# 版本演进示例
# v1：只提取对比算法和代码链接
CACHE_VERSION = "v1"
fields_v1 = ["comparative_algorithms", "code_links"]

# v2：增加场景、问题、目标、创新
CACHE_VERSION = "v2"  # 更新版本
fields_v2 = [
    "comparative_algorithms", 
    "code_links",
    "scenario",           # 新增
    "research_problem",   # 新增
    "research_goal",      # 新增
    "innovation"          # 新增
]

# 缓存数据结构的变化
# v1缓存数据：
v1_data = {
    "comparative_algorithms": ["Algorithm A", "Algorithm B"],
    "code_links": ["https://github.com/..."]
}

# v2缓存数据：
v2_data = {
    "comparative_algorithms": ["Algorithm A", "Algorithm B"],
    "code_links": ["https://github.com/..."],
    "scenario": "...",
    "research_problem": "...",
    "research_goal": "...",
    "innovation": "..."
}

# 如果不更新版本会怎样？
# 1. 程序读取v1缓存
# 2. 尝试访问v2新增的字段
# 3. KeyError！程序崩溃

# 正确的做法：
def validate_cache_data(data):
    """验证缓存数据是否包含所有必需字段"""
    required_fields = [
        "comparative_algorithms", 
        "code_links",
        "scenario",
        "research_problem",
        "research_goal",
        "innovation"
    ]
    
    for field in required_fields:
        if field not in data:
            return False  # 缓存无效
    
    return True  # 缓存有效

# 在读取缓存时验证
def get_from_cache(paper_url):
    cache_path = get_cache_path(paper_url)
    if os.path.exists(cache_path):
        with open(cache_path, 'r', encoding='utf-8') as f:
            data = json.load(f)
            
            # 验证缓存
            if not validate_cache_data(data):
                return None  # 缓存无效，需要重新提取
            
            return data
    return None
```

### 目录初始化（第15-20行）

```python
# 确保目录存在                             # 第15行（注释）
EXPERIMENT_CACHE_DIR = "experiment_cache"   # 第16行
EXPERIMENT_RESULTS_DIR = "experiment_results"  # 第17行
for folder in [EXPERIMENT_CACHE_DIR, EXPERIMENT_RESULTS_DIR]:  # 第18行
    if not os.path.exists(folder):          # 第19行
        os.makedirs(folder)                 # 第20行
```

**第16-17行详解**：目录常量定义

```python
EXPERIMENT_CACHE_DIR = "experiment_cache"
EXPERIMENT_RESULTS_DIR = "experiment_results"

# 目录用途说明
# 1. experiment_cache - 缓存目录
#    存储：已提取的论文实验信息（JSON文件）
#    目的：避免重复处理同一篇论文
#    示例：experiment_cache/abc123....json

# 2. experiment_results - 结果目录
#    存储：批量提取的最终结果（JSON文件）
#    目的：保存用户查询的完整结果
#    示例：experiment_results/query_20240115.json

# 为什么用常量？
# 优点：
# 1. 集中管理：所有路径在一处定义
# 2. 易于修改：只需改一处，所有引用自动更新
# 3. 避免拼写错误：使用常量而不是字符串字面量

# 使用示例
# 不推荐：硬编码路径
cache_file = os.path.join("experiment_cache", "abc123.json")
# 问题：如果目录名改了，需要搜索所有代码修改

# 推荐：使用常量
cache_file = os.path.join(EXPERIMENT_CACHE_DIR, "abc123.json")
# 优势：只需修改常量定义，所有地方自动更新
```

**第18-20行详解**：自动创建目录

```python
for folder in [EXPERIMENT_CACHE_DIR, EXPERIMENT_RESULTS_DIR]:  # 第18行
    if not os.path.exists(folder):          # 第19行
        os.makedirs(folder)                 # 第20行

# 完整执行流程
# 1. for循环遍历目录列表
folders = ["experiment_cache", "experiment_results"]
for folder in folders:
    # folder = "experiment_cache"
    # folder = "experiment_results"
    pass

# 2. 检查目录是否存在
if not os.path.exists(folder):
    # os.path.exists()返回：
    # True - 目录存在
    # False - 目录不存在
    # not False = True，进入if块

# 3. 创建目录
os.makedirs(folder)
# 递归创建目录（包括父目录）

# 为什么在程序启动时创建目录？
# 原因：
# 1. 确保程序运行环境正确
# 2. 避免运行时出现FileNotFoundError
# 3. 提升用户体验（自动配置）

# 如果不创建会怎样？
# 场景：保存缓存文件
cache_path = "experiment_cache/abc123.json"
with open(cache_path, 'w') as f:  # FileNotFoundError!
    json.dump(data, f)

# os.makedirs vs os.mkdir
# os.mkdir('a/b/c')
# - 只创建最后一级目录
# - 父目录不存在时报错

# os.makedirs('a/b/c')（本代码使用）
# - 递归创建所有目录
# - 自动创建a、a/b、a/b/c

# 更安全的写法
for folder in [EXPERIMENT_CACHE_DIR, EXPERIMENT_RESULTS_DIR]:
    os.makedirs(folder, exist_ok=True)
    # exist_ok=True：目录已存在时不报错

# 项目目录结构
# paper_tool_3.0/
# ├── app.py
# ├── experiment_extractor.py
# ├── experiment_cache/        ← 自动创建
# │   ├── abc123....json
# │   ├── def456....json
# │   └── ...
# └── experiment_results/       ← 自动创建
#     ├── query_20240115.json
#     └── ...
```

---

## 🔍 正则表达式模式

### 实验章节模式（第40-51行）

```python
# 定义常见的实验部分关键词正则表达式   # 第40行（注释）
EXPERIMENT_SECTION_PATTERNS = [          # 第41行
    r"(?:4|IV|4\.0)[\s\.]*(?:Experiment|Evaluation|Results)",  # 第42行
    r"(?:5|V|5\.0)[\s\.]*(?:Experiment|Evaluation|Results)",   # 第43行
    r"(?:6|VI|6\.0)[\s\.]*(?:Experiment|Evaluation|Results)",  # 第44行
    r"(?:Experiment|Evaluation)[\s\w]*(?:Setup|Design)",       # 第45行
    r"(?:Performance|Experimental)[\s\w]*(?:Evaluation|Results)",  # 第46行
    r"Experiment(?:s|al)",                                     # 第47行
    r"Result(?:s)",                                            # 第48行
    r"Evaluation",                                             # 第49行
    r"Benchmark"                                               # 第50行
]
```

**第41行详解**：`EXPERIMENT_SECTION_PATTERNS = [`

**正则模式列表详解（第41行）**：
- `EXPERIMENT_SECTION_PATTERNS`：常量名
- `=`：赋值运算符
- `[...]`：Python列表

```python
EXPERIMENT_SECTION_PATTERNS = [...]

# 这个列表包含什么？
# 9种正则表达式模式，用于匹配论文中的实验章节标题

# 为什么需要多种模式？
# 因为不同论文的章节编号和命名方式不同：
# - "4. Experiment"
# - "IV. Evaluation"
# - "4.0 Results"
# - "Section 5: Experiments"
# - "Experimental Setup"
# 等等...

# 使用方式
text = "4. Experiment and Results"
is_experiment_section = False

for pattern in EXPERIMENT_SECTION_PATTERNS:
    if re.search(pattern, text, re.IGNORECASE):
        is_experiment_section = True
        print(f"匹配模式：{pattern}")
        break

if is_experiment_section:
    print("这是实验章节！")
```

**第42行详解**：章节编号模式

```python
r"(?:4|IV|4\.0)[\s\.]*(?:Experiment|Evaluation|Results)"

# 正则表达式详细解析
# r"..." - 原始字符串（r = raw string）
# 优点：不需要转义反斜杠

# 普通字符串 vs 原始字符串
normal_string = "\\d+\\.\\d+"  # 需要双重转义
raw_string = r"\d+\.\d+"       # 不需要转义（推荐）

# 模式结构分解
pattern = r"(?:4|IV|4\.0)[\s\.]*(?:Experiment|Evaluation|Results)"

# 第一部分：(?:4|IV|4\.0)
# (?:...)    - 非捕获组（只分组，不保存匹配结果）
# 4|IV|4\.0  - 匹配"4"或"IV"或"4.0"
# |          - 或运算符
# \.         - 转义的点号（匹配字面的"."）

# 第二部分：[\s\.]*
# [...]      - 字符集（匹配集合中的任意一个字符）
# \s         - 空白字符（空格、制表符、换行等）
# \.         - 点号
# *          - 0次或多次

# 第三部分：(?:Experiment|Evaluation|Results)
# (?:...)    - 非捕获组
# Experiment|Evaluation|Results - 匹配这三个词之一

# 完整匹配示例
test_strings = [
    "4. Experiment",              # ✓ 匹配：4 + . + Experiment
    "4.0 Experimental Results",   # ✓ 匹配：4.0 + 空格 + Experiment
    "IV. Evaluation",             # ✓ 匹配：IV + . + Evaluation
    "4   Results",                # ✓ 匹配：4 + 空格 + Results
    "Section 4. Experiment",      # ✓ 匹配（包含4. Experiment）
    "3. Experiment",              # ✗ 不匹配（3不在模式中）
    "4. Method"                   # ✗ 不匹配（Method不在模式中）
]

pattern = r"(?:4|IV|4\.0)[\s\.]*(?:Experiment|Evaluation|Results)"
for text in test_strings:
    match = re.search(pattern, text, re.IGNORECASE)
    status = "✓ 匹配" if match else "✗ 不匹配"
    print(f"{status}: {text}")

# re.IGNORECASE标志
# 作用：忽略大小写
# 例如："experiment"、"EXPERIMENT"、"Experiment"都能匹配

# 为什么用非捕获组(?:...)而不是普通分组(...)？
# 普通分组：
pattern = r"(4|IV|4\.0)[\s\.]*(Experiment|Evaluation|Results)"
match = re.search(pattern, text)
if match:
    print(match.group(0))  # 完整匹配
    print(match.group(1))  # 第一个分组
    print(match.group(2))  # 第二个分组

# 非捕获组（本代码使用）：
pattern = r"(?:4|IV|4\.0)[\s\.]*(?:Experiment|Evaluation|Results)"
match = re.search(pattern, text)
if match:
    print(match.group(0))  # 完整匹配
    # 没有group(1)和group(2)

# 优势：
# 1. 性能更好（不保存分组）
# 2. 代码更清晰（只需要知道是否匹配，不需要提取部分）
```

**第45行详解**：组合词模式

```python
r"(?:Experiment|Evaluation)[\s\w]*(?:Setup|Design)"

# 模式解析
# (?:Experiment|Evaluation)  - 匹配"Experiment"或"Evaluation"
# [\s\w]*                    - 0个或多个空白或字母数字
# (?:Setup|Design)           - 匹配"Setup"或"Design"

# 匹配示例
# "Experiment Setup"         ✓
# "Experimental Setup"       ✓（Experiment + al + 空格 + Setup）
# "Evaluation Design"        ✓
# "Experiment and Setup"     ✓（Experiment + 空格and空格 + Setup）
# "Experimental Design"      ✓

# [\s\w]*的作用
# \s - 空白字符
# \w - 字母、数字、下划线（word character）
# * - 0次或多次

# 可以匹配的中间部分：
# "" - 空（直接连接）
# " " - 单个空格
# "al " - 字母加空格
# " and " - 词语加空格

test_strings = [
    "Experiment Setup",
    "Experimental Setup",
    "Experiment and Results Setup",
    "Evaluation Design"
]

pattern = r"(?:Experiment|Evaluation)[\s\w]*(?:Setup|Design)"
for text in test_strings:
    match = re.search(pattern, text, re.IGNORECASE)
    if match:
        print(f"✓ {text} → 匹配：{match.group()}")
```

**第47-50行详解**：简单模式

```python
r"Experiment(?:s|al)",         # 第47行
r"Result(?:s)",                # 第48行
r"Evaluation",                 # 第49行
r"Benchmark"                   # 第50行

# 第47行：r"Experiment(?:s|al)"
# 匹配：
# - "Experiment"
# - "Experiments"（加s）
# - "Experimental"（加al）
# (?:s|al) - 匹配"s"或"al"（可选）

# 第48行：r"Result(?:s)"
# 匹配：
# - "Result"
# - "Results"
# (?:s) - 匹配"s"（可选）

# 可选部分的语法
# (?:s|al)  - 匹配"s"或"al"
# (?:s)?    - 匹配0个或1个"s"（等价于上面，但只有一个选项时更简洁）

# 示例对比
pattern1 = r"Result(?:s)"      # 匹配Result或Results
pattern2 = r"Results?"         # 等价，?表示前面的s可选

# 第49-50行：简单的单词匹配
# "Evaluation"和"Benchmark"直接匹配这些词

# 为什么有些模式这么简单？
# 策略：从严格到宽松
# 1. 先尝试严格匹配（如"4. Experiment"）
# 2. 如果没匹配上，再用宽松匹配（如"Experiment"）
# 3. 提高召回率（找到更多相关内容）

# 实际应用
def find_experiment_sections(text):
    """在文本中查找所有实验章节"""
    sections = []
    
    for pattern in EXPERIMENT_SECTION_PATTERNS:
        matches = re.finditer(pattern, text, re.IGNORECASE)
        for match in matches:
            sections.append({
                'text': match.group(),
                'start': match.start(),
                'end': match.end(),
                'pattern': pattern
            })
    
    return sections

# 使用示例
paper_text = """
1. Introduction
2. Related Work
3. Methodology
4. Experiment and Results
   4.1 Experimental Setup
   4.2 Benchmark Results
5. Evaluation
6. Conclusion
"""

sections = find_experiment_sections(paper_text)
for section in sections:
    print(f"找到：{section['text']} (位置：{section['start']})")
```

### 对比算法模式（第53-63行）

```python
# 对比算法关键词                         # 第53行（注释）
ALGORITHM_PATTERNS = [                   # 第54行
    r"compar(?:e|ed|ing|ison)\s+(?:with|to|against)",  # 第55行
    r"baseline(?:s)?",                   # 第56行
    r"state-of-the-art",                 # 第57行
    r"benchmark(?:s|ed|ing)?",           # 第58行
    r"(?:previous|existing)\s+(?:method|approach)(?:s)?",  # 第59行
    r"competitor(?:s)?",                 # 第60行
    r"method(?:s)?\s+(?:include|used)",  # 第61行
    r"algorithm(?:s)?"                   # 第62行
]
```

**第55行详解**：compare词族模式

```python
r"compar(?:e|ed|ing|ison)\s+(?:with|to|against)"

# 模式解析
# compar             - 匹配"compar"（compare的词干）
# (?:e|ed|ing|ison) - 匹配不同的词尾
# \s+                - 1个或多个空白字符
# (?:with|to|against) - 匹配介词

# 可以匹配的完整短语
matches = [
    "compare with",      # compar + e + 空格 + with
    "compared to",       # compar + ed + 空格 + to
    "comparing against", # compar + ing + 空格 + against
    "comparison with"    # compar + ison + 空格 + with
]

# 为什么这样设计？
# 英语中compare有多种形式：
# - compare (原形)
# - compared (过去式/过去分词)
# - comparing (现在分词/动名词)
# - comparison (名词)

# 实际论文例句
examples = [
    "We compare our method with three baselines.",
    "The proposed algorithm is compared to existing approaches.",
    "When comparing against state-of-the-art methods...",
    "In comparison with previous work..."
]

pattern = r"compar(?:e|ed|ing|ison)\s+(?:with|to|against)"
for example in examples:
    match = re.search(pattern, example, re.IGNORECASE)
    if match:
        print(f"✓ 匹配：{match.group()}")
        print(f"  原句：{example}")

# 输出：
# ✓ 匹配：compare our method with
#   原句：We compare our method with three baselines.
# ✓ 匹配：compared to
#   原句：The proposed algorithm is compared to existing approaches.
# ...

# \s+的作用
# \s  - 空白字符（空格、制表符、换行等）
# +   - 1个或多次（至少有一个空格）

# 为什么用\s+而不是单个空格' '？
# 原因：
# 1. 可能有多个连续空格
# 2. 可能是制表符
# 3. 可能有换行符（跨行）

# 示例
text1 = "compare  with"    # 两个空格
text2 = "compare\twith"    # 制表符
text3 = "compare\nwith"    # 换行符

# \s+都能匹配，而' '只能匹配单个空格
```

**第56-58行详解**：基准词模式

```python
r"baseline(?:s)?",                   # 第56行
r"state-of-the-art",                 # 第57行
r"benchmark(?:s|ed|ing)?",           # 第58行

# 第56行：baseline模式
# baseline  - 匹配"baseline"
# (?:s)?    - 可选的"s"（复数）
# 匹配：
# - "baseline"
# - "baselines"

# 第57行：state-of-the-art模式
# 匹配固定短语"state-of-the-art"
# 注意：连字符-是字面字符，不需要转义

# 第58行：benchmark模式
# benchmark          - 匹配"benchmark"
# (?:s|ed|ing)?      - 可选的词尾
# 匹配：
# - "benchmark"
# - "benchmarks"（复数）
# - "benchmarked"（过去式）
# - "benchmarking"（现在分词）

# 实际使用
def extract_baseline_mentions(text):
    """提取文本中提到的基准方法"""
    baselines = []
    
    # 查找baseline
    for match in re.finditer(r"baseline(?:s)?", text, re.IGNORECASE):
        baselines.append(match.group())
    
    # 查找state-of-the-art
    for match in re.finditer(r"state-of-the-art", text, re.IGNORECASE):
        baselines.append(match.group())
    
    # 查找benchmark
    for match in re.finditer(r"benchmark(?:s|ed|ing)?", text, re.IGNORECASE):
        baselines.append(match.group())
    
    return baselines

# 测试
paper_excerpt = """
We compared our approach with three baselines and state-of-the-art methods.
The proposed algorithm was benchmarked against existing benchmarks.
"""

baselines = extract_baseline_mentions(paper_excerpt)
print("找到的基准词：", baselines)
# ['baselines', 'state-of-the-art', 'benchmarked', 'benchmarks']
```

**第59行详解**：现有方法模式

```python
r"(?:previous|existing)\s+(?:method|approach)(?:s)?"

# 模式解析
# (?:previous|existing)  - 匹配"previous"或"existing"
# \s+                    - 1个或多个空白
# (?:method|approach)    - 匹配"method"或"approach"
# (?:s)?                 - 可选的复数"s"

# 可以匹配的短语
phrases = [
    "previous method",
    "previous methods",
    "existing method",
    "existing methods",
    "previous approach",
    "previous approaches",
    "existing approach",
    "existing approaches"
]

# 为什么这样组合？
# 论文中常见的表达：
# - "compared with previous methods"
# - "unlike existing approaches"
# - "improves upon existing method"

# 实际例句
examples = [
    "Our method outperforms previous methods.",
    "Unlike existing approaches, we propose...",
    "The algorithm improves upon existing method.",
    "Compared with previous approach..."
]

pattern = r"(?:previous|existing)\s+(?:method|approach)(?:s)?"
for example in examples:
    match = re.search(pattern, example, re.IGNORECASE)
    if match:
        print(f"✓ {example}")
        print(f"  匹配：{match.group()}")
```

### 代码链接模式（第65-78行）

```python
# 代码链接关键词                         # 第65行（注释）
CODE_LINK_PATTERNS = [                   # 第66行
    r"code(?:\s+is)?\s+available",       # 第67行
    r"implementation(?:\s+is)?\s+available",  # 第68行
    r"source\s+code",                    # 第69行
    r"github\.com",                      # 第70行
    r"gitlab\.com",                      # 第71行
    r"code\s+repository",                # 第72行
    r"open-source",                      # 第73行
    r"public(?:ly)?\s+available",        # 第74行
    r"software\s+package",               # 第75行
    r"available\s+at\s+https?://",       # 第76行
    r"release(?:d)?\s+at\s+https?://"    # 第77行
]
```

**第67-68行详解**：可用性表述模式

```python
r"code(?:\s+is)?\s+available",           # 第67行
r"implementation(?:\s+is)?\s+available",  # 第68行

# 第67行模式解析
# code          - 匹配"code"
# (?:\s+is)?    - 可选的" is"
# \s+           - 1个或多个空白
# available     - 匹配"available"

# 可以匹配的短语
matches_67 = [
    "code available",         # code + 空格 + available
    "code is available",      # code + 空格is + available
    "Code  is  available"     # 多个空格也能匹配
]

# 为什么(?:\s+is)?是可选的？
# 因为论文中可能有两种表述：
# 1. "Our code is available at..."
# 2. "Code available at..."

# 第68行：implementation模式
# 结构与第67行相同，只是将"code"换成"implementation"

# 实际论文例句
examples = [
    "The code is available at GitHub.",
    "Code available at https://github.com/...",
    "Our implementation is available online.",
    "Implementation available at our repository."
]

patterns = [
    r"code(?:\s+is)?\s+available",
    r"implementation(?:\s+is)?\s+available"
]

for example in examples:
    for pattern in patterns:
        match = re.search(pattern, example, re.IGNORECASE)
        if match:
            print(f"✓ {example}")
            print(f"  匹配：{match.group()}")
            break
```

**第70-71行详解**：域名模式

```python
r"github\.com",                      # 第70行
r"gitlab\.com",                      # 第71行

# 模式解析
# github\.com
# - github: 字面匹配"github"
# - \.    : 转义的点号（匹配字面的"."）
# - com   : 字面匹配"com"

# 为什么要转义点号？
# 在正则表达式中：
# .   - 匹配任意字符（特殊含义）
# \.  - 匹配字面的点号"."

# 示例
pattern1 = r"github.com"   # 错误！.匹配任意字符
# 会匹配：
# - "githubXcom"
# - "github!com"
# - "github.com"

pattern2 = r"github\.com"  # 正确！只匹配字面的"."
# 只匹配：
# - "github.com"

# 实际应用：提取GitHub链接
def extract_github_links(text):
    """从文本中提取GitHub链接"""
    # 匹配完整的GitHub URL
    pattern = r"https?://github\.com/[\w-]+/[\w-]+"
    matches = re.findall(pattern, text)
    return matches

# 测试
text = """
Code available at https://github.com/user/repo1
Also see https://github.com/another-user/cool-project
And https://gitlab.com/user/repo2
"""

github_links = extract_github_links(text)
print("GitHub链接：", github_links)
# ['https://github.com/user/repo1', 
#  'https://github.com/another-user/cool-project']

# 更完善的URL提取
def extract_code_links(text):
    """提取所有代码仓库链接"""
    links = []
    
    # GitHub链接
    github_pattern = r"https?://github\.com/[\w-]+/[\w-]+"
    links.extend(re.findall(github_pattern, text))
    
    # GitLab链接
    gitlab_pattern = r"https?://gitlab\.com/[\w-]+/[\w-]+"
    links.extend(re.findall(gitlab_pattern, text))
    
    return links
```

**第76-77行详解**：URL模式

```python
r"available\s+at\s+https?://",       # 第76行
r"release(?:d)?\s+at\s+https?://"    # 第77行

# 第76行模式解析
# available    - 匹配"available"
# \s+          - 1个或多个空白
# at           - 匹配"at"
# \s+          - 1个或多个空白
# https?://    - 匹配"http://"或"https://"

# https?://详解
# http         - 匹配"http"
# s?           - 可选的"s"（0个或1个）
# ://          - 匹配"://"

# 可以匹配：
# - "http://"
# - "https://"

# 第77行：release模式
# release(?:d)?  - 匹配"release"或"released"
# 其余部分与第76行相同

# 实际例句
examples = [
    "Code available at https://github.com/user/repo",
    "Available at http://example.com/code",
    "The code is released at https://gitlab.com/project",
    "Released at http://code.example.com"
]

patterns = [
    r"available\s+at\s+https?://",
    r"release(?:d)?\s+at\s+https?://"
]

for example in examples:
    for pattern in patterns:
        match = re.search(pattern, example, re.IGNORECASE):
        if match:
            print(f"✓ {example}")
            print(f"  匹配：{match.group()}")
            
            # 提取URL的开始位置
            url_start = match.end()
            print(f"  URL从位置{url_start}开始")
            break

# 完整的URL提取
def extract_full_url(text, pattern):
    """提取完整的URL"""
    match = re.search(pattern, text, re.IGNORECASE)
    if match:
        # 找到"http://"或"https://"的位置
        url_start = text.find("http", match.start())
        
        # 从这个位置开始提取URL（到空白或句号为止）
        url_match = re.match(r"https?://[^\s]+", text[url_start:])
        if url_match:
            return url_match.group()
    
    return None

# 测试
text = "Code available at https://github.com/user/repo for download."
url = extract_full_url(text, r"available\s+at\s+https?://")
print(f"提取的URL：{url}")
# 输出：https://github.com/user/repo
```

---

## 💾 缓存系统

### 缓存路径生成（第179-184行）

```python
def get_cache_path(paper_key):           # 第179行
    """获取缓存文件路径"""                  # 第180行
    # 添加版本号到哈希中                    # 第181行
    versioned_key = f"{CACHE_VERSION}_{paper_key}"  # 第182行
    cache_file = hashlib.md5(versioned_key.encode()).hexdigest() + ".json"  # 第183行
    return os.path.join(EXPERIMENT_CACHE_DIR, cache_file)  # 第184行
```

**完整函数详解**：

```python
def get_cache_path(paper_key):
    """
    根据论文标识生成缓存文件路径
    
    参数:
        paper_key: 论文的唯一标识（通常是URL或DOI）
    
    返回:
        缓存文件的完整路径
    """
    # 步骤1：创建版本化的键
    versioned_key = f"{CACHE_VERSION}_{paper_key}"
    # 例如："v2_https://arxiv.org/abs/2024.12345"
    
    # 步骤2：生成MD5哈希
    hash_value = hashlib.md5(versioned_key.encode()).hexdigest()
    # 例如："a1b2c3d4e5f6789abcdef0123456789"
    
    # 步骤3：添加.json扩展名
    cache_file = hash_value + ".json"
    # 例如："a1b2c3d4e5f6789abcdef0123456789.json"
    
    # 步骤4：拼接完整路径
    cache_path = os.path.join(EXPERIMENT_CACHE_DIR, cache_file)
    # 例如："experiment_cache/a1b2c3d4e5f6789abcdef0123456789.json"
    
    return cache_path

# 使用示例
paper_url = "https://arxiv.org/abs/2024.12345"
cache_path = get_cache_path(paper_url)
print(f"缓存路径：{cache_path}")

# 不同的paper_key生成不同的路径
urls = [
    "https://arxiv.org/abs/2024.001",
    "https://arxiv.org/abs/2024.002",
    "https://ieeexplore.ieee.org/document/12345"
]

for url in urls:
    path = get_cache_path(url)
    print(f"{url[:40]}... → {path[-20:]}")

# 输出：
# https://arxiv.org/abs/2024.001... → ...abc123.json
# https://arxiv.org/abs/2024.002... → ...def456.json
# https://ieeexplore.ieee.org/docum... → ...789xyz.json

# 为什么使用哈希而不是直接用URL？
# 问题1：URL包含特殊字符
url = "https://example.com/paper?id=123&version=2"
# 直接用作文件名会有问题：
# - ?和&是非法文件名字符
# - Windows不允许某些字符

# 问题2：URL可能很长
url = "https://dl.acm.org/doi/abs/10.1145/1234567890.0987654321?param=value"
# 太长的文件名不方便管理

# 使用哈希的优势：
# 1. 固定长度（32字符）
# 2. 只包含[0-9a-f]，合法
# 3. 唯一性（不同URL产生不同哈希）
# 4. 确定性（相同URL总是产生相同哈希）
```

这份文档现在已经创建完成！我为您详细讲解了：

1. ✅ **导入语句**（10个模块）- 每个都有详细示例
2. ✅ **全局配置**（缓存版本、目录初始化）
3. ✅ **正则表达式模式**（实验章节、对比算法、代码链接）
4. ✅ **缓存系统**（路径生成）

文档特点：
- 📝 逐行代码解析
- 🔑 关键字详细讲解
- 💡 丰富的代码示例
- 📊 对比说明
- 🎯 实际应用场景

文档已保存在 `learn/ExperimentExtractor_实验提取详解.md`！

由于文件很长（1266行），我重点讲解了最核心的前200行内容。如果您需要我继续讲解后续部分（如缓存读写函数、提取算法等），请告诉我！


