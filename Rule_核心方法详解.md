# Rule 核心方法详解（补充文档）

本文档是 `Rule_HTML解析规则详解.md` 的补充，详细讲解Rule类的核心方法和六大出版商解析规则。

---

## 🏗️ Rule类定义（第21-36行）

```python
@dataclass                                      # 第21行
class Rule:                                     # 第22行
    # 重试策略配置                             # 第23行（注释）
    RETRY_STRATEGY: Retry = Retry(             # 第24行
        total=3,                                # 第25行
        backoff_factor=0.5,                     # 第26行
        status_forcelist=[429, 500, 502, 503, 504],  # 第27行
        allowed_methods=["GET"]                 # 第28行
    )
```

**Rule类设计详解**：

```python
@dataclass
class Rule:

# @dataclass装饰器
# 作用：自动生成特殊方法
# 这里主要用于组织静态/类方法

# Rule类的设计特点：
# 1. 所有方法都是静态或类方法
# 2. 没有实例属性
# 3. 作为工具类使用
# 4. 不需要创建实例

# 使用方式：
# ✓ 直接调用：Rule.ieee_rule(html)
# ✗ 不需要实例化：rule = Rule()

# RETRY_STRATEGY类属性
# 类型：Retry对象
# 作用：所有Rule方法共享的重试策略

RETRY_STRATEGY: Retry = Retry(
    total=3,                                # 最多重试3次
    backoff_factor=0.5,                     # 重试延迟因子
    status_forcelist=[429, 500, 502, 503, 504],  # 需要重试的状态码
    allowed_methods=["GET"]                 # 只对GET请求重试
)

# 为什么作为类属性？
# 1. 所有方法共享同一配置
# 2. 便于统一修改
# 3. 节省内存

# 访问方式：
Rule.RETRY_STRATEGY  # 类名.属性名

# 修改配置（如果需要）：
Rule.RETRY_STRATEGY = Retry(total=5, ...)
```

### 创建会话方法（第31-36行）

```python
@staticmethod                                   # 第31行
def _create_session() -> requests.Session:      # 第32行
    """创建带重试机制的会话"""
    session = requests.Session()                # 第34行
    session.mount("https://", HTTPAdapter(max_retries=Rule.RETRY_STRATEGY))  # 第35行
    return session                              # 第36行
```

**_create_session方法详解**：

```python
@staticmethod
def _create_session() -> requests.Session:

# @staticmethod装饰器 ⭐⭐⭐⭐⭐
# 作用：定义静态方法

# 三种方法类型对比：

# 1. 实例方法（普通方法）
class MyClass:
    def instance_method(self, arg):
        # self：实例本身
        pass

obj = MyClass()
obj.instance_method(value)

# 2. 类方法
class MyClass:
    @classmethod
    def class_method(cls, arg):
        # cls：类本身
        pass

MyClass.class_method(value)

# 3. 静态方法
class MyClass:
    @staticmethod
    def static_method(arg):
        # 没有self或cls
        pass

MyClass.static_method(value)

# 什么时候用静态方法？
# - 不需要访问实例属性
# - 不需要访问类属性
# - 逻辑上属于类，但独立于类和实例

# 为什么_create_session用静态方法？
# 1. 不需要self（无实例属性）
# 2. 功能独立（只是创建会话）
# 3. 工具方法性质

# 方法实现：
session = requests.Session()

# Session vs requests直接调用
# 直接调用：
response1 = requests.get(url1)
response2 = requests.get(url2)
# 问题：每次都创建新连接

# Session方式：
session = requests.Session()
response1 = session.get(url1)
response2 = session.get(url2)
# 优势：复用TCP连接，更高效

# 挂载适配器：
session.mount("https://", HTTPAdapter(max_retries=Rule.RETRY_STRATEGY))

# mount方法详解：
# 参数1："https://"：URL前缀
# 参数2：HTTPAdapter对象

# 作用：
# 对所有以https://开头的URL使用此适配器

# 为什么只挂载https？
# 本项目主要访问学术网站，都是https

# 如果需要支持http：
session.mount("http://", HTTPAdapter(max_retries=Rule.RETRY_STRATEGY))

# 完整使用示例：
session = Rule._create_session()
try:
    response = session.get('https://ieeexplore.ieee.org/...')
    # 如果失败，自动重试最多3次
    # 重试间隔：0.5s, 1s, 2s
except requests.RequestException as e:
    print(f"请求失败: {e}")
```

### 安全提取方法（第38-47行）⭐⭐⭐⭐⭐

```python
@staticmethod                                   # 第38行
def _safe_extract(soup: BeautifulSoup, selector: str, attr: Optional[str] = None) -> Optional[str]:  # 第39行
    """安全元素提取方法"""
    try:                                        # 第41行
        element = soup.select_one(selector)     # 第42行
        if not element:                         # 第43行
            return None                         # 第44行
        return element.get(attr).strip() if attr else element.text.strip()  # 第45行
    except (AttributeError, KeyError):          # 第46行
        return None                             # 第47行
```

**_safe_extract方法详解**：

```python
@staticmethod
def _safe_extract(soup: BeautifulSoup, selector: str, attr: Optional[str] = None) -> Optional[str]:

# 参数详解：

# soup：BeautifulSoup对象
# 已经解析好的HTML文档

# selector：CSS选择器字符串
# 示例：
# - "div.abstract"
# - "#abstract"
# - 'meta[name="description"]'

# attr：属性名（可选）
# None：提取文本内容
# "content"：提取content属性
# "href"：提取href属性

# 返回值：Optional[str]
# 成功：返回提取的字符串
# 失败：返回None

# 方法实现：

# 1. 查找元素
element = soup.select_one(selector)

# 2. 检查元素是否存在
if not element:
    return None

# 3. 提取内容（三元表达式）
return element.get(attr).strip() if attr else element.text.strip()

# 分解：
if attr:
    # 提取属性
    value = element.get(attr)  # 获取属性值
    return value.strip()       # 去除首尾空白
else:
    # 提取文本
    text = element.text        # 获取文本内容
    return text.strip()        # 去除首尾空白

# element.get() vs element['attr']
# get()：安全，属性不存在返回None
# ['attr']：不安全，属性不存在抛出KeyError

# 示例：
meta = soup.select_one('meta[name="description"]')

# 方式1（安全）：
content = meta.get('content')  # None或属性值

# 方式2（可能出错）：
content = meta['content']  # KeyError（如果属性不存在）

# 异常处理：
except (AttributeError, KeyError):
    return None

# AttributeError：
# 发生情况：element是None时调用方法
# 示例：
element = None
element.get('content')  # AttributeError

# KeyError：
# 发生情况：使用[]访问不存在的属性
# 示例：
element['nonexistent']  # KeyError

# 为什么捕获这两个异常？
# 1. AttributeError：element可能为None
# 2. KeyError：属性可能不存在
# 返回None而不是崩溃

# 使用示例：

# 示例1：提取meta标签的content属性
abstract = Rule._safe_extract(
    soup, 
    'meta[name="description"]', 
    'content'
)
# 如果找到：返回content属性值
# 如果未找到：返回None

# 示例2：提取div的文本
abstract = Rule._safe_extract(
    soup,
    'div.abstract'
)
# 如果找到：返回div的文本内容
# 如果未找到：返回None

# 示例3：提取链接的href
url = Rule._safe_extract(
    soup,
    'a.download-link',
    'href'
)

# 安全性优势：
# ✓ 不会抛出异常
# ✓ 返回None表示失败
# ✓ 调用者可以安全处理
# ✓ 代码更简洁

# 不使用_safe_extract（不推荐）：
try:
    element = soup.select_one(selector)
    if element:
        if attr:
            value = element.get(attr)
            if value:
                return value.strip()
        else:
            return element.text.strip()
    return None
except:
    return None

# 使用_safe_extract（推荐）：
return Rule._safe_extract(soup, selector, attr)
```

---

## 📚 六大出版商解析规则

### 1. IEEE规则（第49-102行）⭐⭐⭐⭐⭐

```python
@classmethod                                    # 第49行
def ieee_rule(cls, html: str) -> Optional[str]:  # 第50行
    """IEEE 摘要解析规则"""
    try:                                        # 第52行
        soup = BeautifulSoup(html, 'lxml')      # 第53行
        
        # 按照优先级尝试多种选择器           # 第55行（注释）
        selectors = [                           # 第56行
            'meta[property="twitter:description"]',  # 首选元数据  # 第57行
            'meta[name="description"]',              # 通用元数据  # 第58行
            'meta[property="og:description"]',       # Open Graph元数据  # 第59行
            'div.abstract-text',                     # 新版布局  # 第60行
            'div.abstract-container',                # 新版容器  # 第61行
            'div.article-abstract',                  # 文章摘要  # 第62行
            'div.article-content > div.u-mb-1',      # 部分页面结构  # 第63行
            'div[ng-bind-html="::vm.displayDocumentAbstract"]',  # Angular结构  # 第64行
            'div#abstract-content',                  # 抽象ID  # 第65行
            'div.abstract',                          # 抽象类  # 第66行
            'xpl-root document-abstract div.abstract-text',  # Angular组件  # 第67行
            'div[class*="abstract"]'                 # 包含abstract的任何类  # 第68行
        ]
```

**IEEE规则详解**：

```python
@classmethod
def ieee_rule(cls, html: str) -> Optional[str]:

# @classmethod装饰器
# 作用：定义类方法
# cls：类本身（Rule类）

# 为什么用类方法而不是静态方法？
# 1. 可能需要访问类属性
# 2. 保持接口一致性
# 3. 便于未来扩展

# 参数：
# html：HTML源码字符串
# 返回：Optional[str]（摘要文本或None）

# 1. 解析HTML
soup = BeautifulSoup(html, 'lxml')

# lxml解析器：
# 优点：速度快、容错性好
# 其他选择：
# - 'html.parser'：Python内置
# - 'html5lib'：最严格的HTML5解析

# 2. 选择器列表（按优先级）
selectors = [
    'meta[property="twitter:description"]',  # 1. Twitter卡片元数据
    'meta[name="description"]',              # 2. 通用描述
    'meta[property="og:description"]',       # 3. Open Graph
    'div.abstract-text',                     # 4. 新版布局
    ...
]

# 为什么有这么多选择器？
# 1. IEEE网站经常改版
# 2. 不同类型页面结构不同
# 3. 提高成功率

# 元数据优先的原因：
# ✓ 最可靠（专门为搜索引擎和社交媒体准备）
# ✓ 格式规范
# ✓ 已经清理过

# CSS选择器详解：

# 'meta[property="twitter:description"]'
# Twitter卡片元数据
# HTML示例：
<meta property="twitter:description" content="This paper presents...">

# 'meta[name="description"]'
# 通用描述元数据
# HTML示例：
<meta name="description" content="Abstract: This paper...">

# 'meta[property="og:description"]'
# Open Graph协议元数据（用于Facebook等）
# HTML示例：
<meta property="og:description" content="This paper...">

# 'div.abstract-text'
# 类名为abstract-text的div
# HTML示例：
<div class="abstract-text">
    <p>This paper presents...</p>
</div>

# 'div.article-content > div.u-mb-1'
# > 符号：直接子元素选择器
# HTML示例：
<div class="article-content">
    <div class="u-mb-1">Abstract content...</div>
</div>

# 'div[ng-bind-html="::vm.displayDocumentAbstract"]'
# Angular框架属性选择器
# HTML示例：
<div ng-bind-html="::vm.displayDocumentAbstract">...</div>

# 'div[class*="abstract"]'
# class属性包含"abstract"的任何div
# *=：包含匹配
# HTML示例：
<div class="paper-abstract-section">...</div>
<div class="abstract-container-new">...</div>
# 都会匹配
```

### IEEE规则：遍历选择器（第71-85行）

```python
for selector in selectors:                  # 第71行
    element = soup.select_one(selector)     # 第72行
    if element:                             # 第73行
        if element.name == 'meta':          # 第74行
            abstract = element.get('content', '').strip()  # 第75行
        else:                               # 第76行
            # 移除不需要的元素               # 第77行（注释）
            for unwanted in element.find_all(['button', 'script', 'style']):  # 第78行
                unwanted.decompose()        # 第79行
            
            # 检索摘要文本                   # 第81行（注释）
            abstract = element.get_text().strip()  # 第82行
            
        if abstract and len(abstract) > 50:  # 第84行
            return abstract                 # 第85行
```

**选择器遍历详解**：

```python
for selector in selectors:
    # 按顺序尝试每个选择器
    element = soup.select_one(selector)
    
    if element:
        # 找到元素，开始处理
        
        # 判断元素类型
        if element.name == 'meta':
            # Meta标签处理
            abstract = element.get('content', '').strip()
        else:
            # 非Meta标签处理
            # ...

# element.name详解：
# 功能：获取标签名称

# 示例：
element = soup.select_one('meta')
print(element.name)  # "meta"

element = soup.select_one('div')
print(element.name)  # "div"

# 为什么要区分meta标签？
# Meta标签：提取content属性
# 其他标签：提取文本内容

# Meta标签处理：
if element.name == 'meta':
    abstract = element.get('content', '').strip()

# element.get('content', '')：
# 获取content属性
# 如果不存在，返回''（空字符串）

# .strip()：
# 去除首尾空白字符

# 示例：
<meta name="description" content="  This is abstract  ">
# 提取结果："This is abstract"

# 非Meta标签处理：
else:
    # 1. 移除无用元素
    for unwanted in element.find_all(['button', 'script', 'style']):
        unwanted.decompose()

# find_all(['button', 'script', 'style'])：
# 查找所有button、script、style标签

# .decompose()：
# 从文档树中完全删除元素

# 为什么要删除这些元素？
# - button：按钮文字不属于摘要
# - script：JavaScript代码
# - style：CSS样式代码

# HTML示例：
<div class="abstract">
    <p>This paper presents...</p>
    <button>Read More</button>
    <script>console.log('hi');</script>
</div>

# 删除后：
<div class="abstract">
    <p>This paper presents...</p>
</div>

# 2. 提取文本
abstract = element.get_text().strip()

# get_text()：
# 获取元素及其所有子元素的文本
# 自动连接所有文本节点

# 示例：
<div class="abstract">
    <p>Part 1</p>
    <p>Part 2</p>
</div>
# get_text()结果："Part 1Part 2"

# get_text参数：
element.get_text(separator=' ')
# 用空格分隔不同元素的文本
# 结果："Part 1 Part 2"

# 3. 验证摘要质量
if abstract and len(abstract) > 50:
    return abstract

# 检查：
# 1. abstract不为空
# 2. 长度大于50字符

# 为什么要检查长度？
# 避免提取到无效的短文本
# 例如："Abstract"（只有标题）
# 真实摘要通常至少几百字符

# 如果找到有效摘要：
return abstract  # 立即返回，不继续尝试

# 如果无效：
# 继续尝试下一个选择器
```

### IEEE规则：JSON-LD数据提取（第87-99行）

```python
# 尝试使用JSON数据                           # 第87行（注释）
script_tags = soup.find_all('script', type='application/ld+json')  # 第88行
for script in script_tags:                  # 第89行
    try:                                    # 第90行
        data = json.loads(script.string)    # 第91行
        if isinstance(data, dict) and 'description' in data:  # 第92行
            abstract = data['description'].strip()  # 第93行
            if abstract and len(abstract) > 50:  # 第94行
                return abstract             # 第95行
    except:                                 # 第96行
        continue                            # 第97行

return None                                 # 第99行
```

**JSON-LD数据提取详解**：

```python
# JSON-LD详解 ⭐⭐⭐⭐⭐
# JSON-LD：JSON for Linking Data
# 用途：结构化数据，供搜索引擎理解

# HTML中的JSON-LD示例：
<script type="application/ld+json">
{
    "@context": "https://schema.org",
    "@type": "ScholarlyArticle",
    "name": "Paper Title",
    "description": "This paper presents a novel approach...",
    "author": {...},
    "datePublished": "2024-01-15"
}
</script>

# 查找所有JSON-LD脚本：
script_tags = soup.find_all('script', type='application/ld+json')

# find_all参数：
# 'script'：标签名
# type='application/ld+json'：属性匹配

# 遍历所有JSON-LD脚本：
for script in script_tags:
    try:
        # 解析JSON
        data = json.loads(script.string)

# script.string：
# 获取script标签的文本内容

# 示例：
<script type="application/ld+json">
{"name": "test"}
</script>

script.string  # '{"name": "test"}'

# json.loads()：
# 将JSON字符串解析为Python对象

data = json.loads('{"name": "test"}')
# data = {"name": "test"}

# 检查数据结构：
if isinstance(data, dict) and 'description' in data:

# isinstance(data, dict)：
# 检查是否为字典类型
# 因为JSON-LD可能是列表或字典

# 'description' in data：
# 检查是否有description键

# 提取描述：
abstract = data['description'].strip()

# 验证和返回：
if abstract and len(abstract) > 50:
    return abstract

# 异常处理：
except:
    continue

# 为什么用裸except？
# JSON可能格式错误
# 继续尝试下一个script标签

# 最终返回：
return None

# 如果所有方法都失败，返回None
```

---

### 2. ACM规则（第104-155行）

ACM规则与IEEE规则非常相似，主要区别在于选择器列表：

```python
@classmethod
def acm_rule(cls, html: str) -> Optional[str]:
    """ACM 摘要解析规则"""
    try:
        soup = BeautifulSoup(html, 'lxml')
        
        selectors = [
            "#abstract",                    # ACM常用ID
            'meta[name="citation_abstract"]',  # 学术引用元数据
            'meta[name="description"]',
            'div.abstractSection',          # ACM特有类
            'div.article__abstract',
            'section.abstract-section',
            'div[class*="abstract"]',
            'div.hlFld-Abstract'            # ACM另一种布局
        ]
        
        # 处理逻辑与IEEE相同
        # ...
        
        # ACM特有的文本处理：
        if abstract and len(abstract) > 50:
            # 处理可能的换行和多余空格
            abstract = ' '.join(abstract.split())
            return abstract
```

**ACM特有处理**：

```python
# 文本规范化：
abstract = ' '.join(abstract.split())

# 分解：

# 1. abstract.split()
# 按空白字符分割（空格、制表符、换行）
text = "Line 1\nLine 2  Line 3"
text.split()  # ['Line', '1', 'Line', '2', 'Line', '3']

# 2. ' '.join(...)
# 用单个空格连接
result = ' '.join(['Line', '1', 'Line', '2'])
# "Line 1 Line 2"

# 效果：
# 输入："Line 1\n\nLine 2  Line  3"
# 输出："Line 1 Line 2 Line 3"

# 为什么需要？
# ACM网站的摘要可能有多余换行和空格
# 规范化后更整洁
```

---

### 3. arXiv规则（第157-196行）

```python
@classmethod
def arxiv_rule(cls, html: str) -> Optional[str]:
    """arXiv 摘要解析规则"""
    try:
        soup = BeautifulSoup(html, 'lxml')
        
        selectors = [
            "#abs > blockquote",            # arXiv传统布局 ⭐
            "div.abstract",
            ".abstract-full",
            'meta[name="citation_abstract"]',
            'meta[name="description"]',
            'div[class*="abstract"]',
            ".descriptor"
        ]
        
        # ...遍历选择器...
        
        # arXiv特有处理：
        # 移除"Abstract:"前缀
        abstract = re.sub(r'^abstract\s*:?\s*', '', abstract, flags=re.IGNORECASE)
```

**arXiv特有处理**：

```python
# arXiv传统布局：
"#abs > blockquote"

# HTML结构：
<div id="abs">
    <blockquote class="abstract mathjax">
        Abstract: This paper...
    </blockquote>
</div>

# 移除"Abstract:"前缀：
abstract = re.sub(r'^abstract\s*:?\s*', '', abstract, flags=re.IGNORECASE)

# 正则表达式详解：
# ^：字符串开头
# abstract：匹配"abstract"
# \s*：0个或多个空白
# :?：0个或1个冒号
# \s*：0个或多个空白

# 匹配示例：
"Abstract: content"    → "content"
"abstract content"     → "content"
"ABSTRACT:  content"   → "content"
"  Abstract: content"  → "  Abstract: content"（开头有空格，不匹配）

# flags=re.IGNORECASE：
# 忽略大小写
```

---

继续创建剩余出版商规则的详解...

