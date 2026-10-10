# Lab 1 实验报告

## 学生信息

- **姓名：陈俊宏**
- **学号：112404260127**
- **班级：数据1241**
- **GitHub 用户名：Crimson06-22**
- **仓库地址：https://github.com/Crimson06-22/bigdata-collection-labs-112404260127**

> 文件位置：`lab01/REPORT.md`  
> 完整代码保存在 `lab1_html_api.ipynb`。  
> 本报告只填写关键发现、关键代码、结果和指定截图，**不要把整份 Notebook 复制进来**。

---

# 任务 1：判断数据加载方式

## 1.1 Device：判断动态加载并找到 API

### 判断结果

- 页面：`/devices`
- 数据加载方式：`动态加载`
- 判断依据：

请用 1～2 句话说明：

1. 在“查看网页源代码”中无法搜索到页面上的设备名称；
2. 在 Network 的 Fetch/XHR 中能找到返回设备数据的请求。

### API 信息

- Request URL：https://115.159.159.82/api/devices?page=1&page_size=50
- Request Method：GET
- Query 参数：page: 1,page_size: 50


### 【截图 1】Device API 发现证据

文件名：

```text
images/01_device_api.png
```

**截图内容必须是浏览器 DevTools，不是 Jupyter 代码。**

截图中应能看清：

1. 浏览器的 **Network** 面板；
2. 已选中真正返回设备列表的 Fetch/XHR 请求；
3. 右侧 **Headers** 中的 Request URL；
4. Query String Parameters（或 URL 中的分页参数）。

> 不要截整个桌面；尽量裁剪到 Network 请求和右侧 Headers 区域。

插入：

![Device API 发现证据](./images/01_device_api.png)

---

## 1.2 Company：判断静态加载

- 页面：`/companies`
- 数据加载方式：`静态加载`
- 判断依据：

请说明：使用“右键 → 查看网页源代码”后，能搜索到页面中真实显示的企业名称。

### 【截图 2】Company 静态加载证据

文件名：

```text
images/02_company_source.png
```

**截图内容必须是“查看网页源代码”页面，不是 Elements，也不是 Jupyter。**

截图中应能看清：

1. 浏览器正在显示 `/companies` 的网页源代码；
2. 使用 `Ctrl + F` 搜索一个页面中真实存在的企业名称；
3. 源代码中能看到该企业名称或对应企业记录。

插入：

![Company 静态加载证据](./images/02_company_source.png)

---

# 任务 2：Device API 分页采集

## 2.1 API 分页结构

填写你实际从 JSON 中观察到的字段：

| 含义   | JSON 字段 |
| ---- | ------- |
| 当前页  | `page`  |
| 每页条数 | `page_size` |
| 设备总数 | `total` |
| 设备列表 | `items` |

---

## 2.2 关键代码

只填写分页采集的核心部分，不要粘贴整个 Notebook。

```python
# 从第一页响应中读取分页信息
total = data["total"]
page_size = data["page_size"]

total_pages = (total + page_size - 1) // page_size

devices = []

for page in range(1, total_pages + 1):
    r = requests.get(
        api_url,
        params={
            "page": page,
            "page_size": page_size
        },
        timeout=10
    )

    page_data = r.json()
    page_items = page_data["items"]
    devices.extend(page_items)
```

---

## 2.3 采集结果

- 每页条数：50
- 设备总数：5000
- 程序计算得到的总页数：100
- 最终采集设备数：5000
- 唯一 `device_id` 数：5000

回答：

> 为什么总页数应由 API 返回的 `total` 和 `page_size` 计算，而不是直接在代码里写死？

答：因为数据总量和每页条数都可能变化。写死页数会在数据增减时导致**漏采**（总数变多时）或**重复请求**（总数变少时），代码也会随之失效。用 `(total + page_size - 1) // page_size` 向上取整计算，可以让程序在任意数据量下都自动采全、且不多采一页。

---

# 任务 3：Company HTML 解析

## 3.1 企业记录 Selector

- 企业记录 selector：`article.company-card`
- 第 1 页企业记录数：20

说明为什么选择这个 selector：

- 主要使用了 `id / class / data-* / 父子层级` 中的哪一种？
- 是否参考过浏览器 Copy selector 或 AI？
- 最终为什么采用现在这个写法？

答：主要使用了 **class** 和 **data-\*** 属性。每条企业记录的最外层节点就是 `<article class="company-card" data-company-id="C000001">`：`company-card` 是语义明确、专门用于标识企业卡片的 class，对页面其他结构变化不敏感；同时该节点自带 `data-company-id` 属性，可直接读取企业 ID，不必再从子节点文本里截取。我先在 Elements 中观察了 DOM，确认所有记录共享该 class，没有使用浏览器「Copy selector」得到的长路径（那会产生大量 `nth-child(...)` 层级，脆弱且难读）。最终选择 `article.company-card` 这一短且稳定的写法。

---

## 3.2 关键代码

只填写“记录定位 + 一条记录的关键字段”。

```python
cards = soup.select("article.company-card")

card = cards[0]

company_id = card.get("data-company-id")
name = card.select_one("h2 a").get_text(strip=True)
status = card.select_one(".badge").get_text(strip=True)
```

行业、区域、设备数量使用 `dt/dd` 结构后，请写出你最终如何得到：

```python
industry = details["所属行业"]
zone = details["所在区域"]
device_count = int(details["设备数量"].replace("台", "").strip())
```

---

## 3.3 字段来源说明

任选 **2 个字段**说明：

1. 字段：`company_id`
   - 来自：`HTML 属性`
   - 使用的方法：`card.get("data-company-id")`，直接读取卡片节点的 `data-*` 属性，不需要解析文本。
2. 字段：`device_count`
   - 来自：`文本`
   - 使用的方法：先在 `dl > div` 中以 `dt` 文本（「设备数量」）为键建立字典，取出对应 `dd` 的文本（形如「6 台」），再用 `.replace("台","").strip()` 去掉单位并转成 `int`。

---

# 任务 4：Company “下一页”分页

## 4.1 下一页元素

- “下一页” selector：`nav.pagination a`，并**按文本内容筛选**出文本为「下一页」的元素
- 第 1 页 `href` 示例：`/companies?page=2`

### 【截图 3】Company “下一页”元素

文件名：

```text
images/03_company_next.png
```

**截图内容必须是浏览器 DevTools 的 Elements 面板。**

截图中应能看清：

1. 页面底部“下一页”对应的 `<a>` 元素被选中；
2. 该元素的 `class`、`href` 等关键属性；
3. 能判断你为什么使用当前 selector。

> 不需要截 Python 代码；这一张图只证明你找到了正确的 DOM 元素。

插入：

![Company 下一页元素](images/03_company_next.png)

---

## 4.2 关键代码

```python
next_link = soup.select_one("nav.pagination a")

next_href = next_link.get("href")

next_url = urljoin(
    current_url,
    next_href
)
```

再用 2～3 句话说明：

> 你的程序如何完成“当前页 → 找下一页 href → 请求下一页 → 再解析”的循环？

答：程序维护一个「当前页 URL」，每轮先请求该 URL 并用 BeautifulSoup 解析出本页全部企业卡片；然后调用 `find_next_link` 精确定位文本为「下一页」的 `<a>` 元素，用 `urljoin` 把它的相对 `href` 拼成绝对 URL，作为下一轮的当前页 URL。如此循环 3 次即可连续采集 3 页；若某页找不到「下一页」元素则提前结束。

---

## 4.3 三页采集结果

| 页码     | 企业数 |
| ------ | --: |
| 1      |  20 |
| 2      |  20 |
| 3      |  20 |
| **合计** | **60** |

- 唯一 `company_id` 数：60

---

# 任务 5：结果验证

## 5.1 最终结果

| 检查项                   | 结果 |
| --------------------- | -: |
| Company 总数            | 60 |
| 唯一 `company_id`       | 60 |
| Device 总数             | 5000 |
| 唯一 `device_id`        | 5000 |
| Company–Device 不一致企业数 | 0 |

### 【截图 4】最终验证结果

文件名：

```text
images/04_validation.png
```

**截图内容必须是 JupyterLab 的运行结果，不是代码截图，也不是浏览器截图。**

截图中应能同时看清以下输出：

```text
Company 总数
唯一 company_id 数
Device 总数
唯一 device_id 数
Company–Device 不一致企业数
```

建议把这几个 `print()` 放在同一个 Notebook 单元格中运行后截图。

插入：

![最终验证结果](images/04_validation.png)

---

## 5.2 一致性检查关键代码

只填写最关键的比较逻辑：

```python
from collections import Counter

device_counts = Counter(
    x["company_id"] for x in devices
)

mismatch_count = 0

for company in companies:
    cid = company["company_id"]
    expected = company["device_count"]
    actual = device_counts.get(cid, 0)

    if expected != actual:
        mismatch_count += 1
```

回答：

> 为什么 `status_code == 200` 仍然不能说明“数据已经采集正确”？

答：`status_code == 200` 只能说明**这一次 HTTP 请求被服务器正常响应**，它不校验返回内容是否完整、是否正确。例如分页参数写错导致只采到第一页、或翻页逻辑有误导致重复采集，请求依然都是 200。要确认数据正确，必须额外做数量校验（总数是否等于 `total`）、唯一性校验（`device_id` / `company_id` 是否重复）以及跨数据源一致性校验（Company 的 `device_count` 与实际 Device 数是否吻合）。

---

# 任务 6：保存采集结果

确认已经生成：

```text
data/companies.json
data/devices.json
```

填写：

- `companies.json`：`已生成`
- `devices.json`：`已生成`

不需要截图 JSON 全文，也不需要把 JSON 内容复制到报告。

---

# 实验结论

用 **80～150 字**总结：

1. 静态加载与动态加载页面的处理方式有什么不同；
2. HTML “下一页”分页和 API `page` 分页有什么不同；
3. 你使用了哪些方法验证采集结果。

答：静态加载与动态加载页面的处理方式不同：`/companies` 的数据随初始 HTML 一起返回，可直接用 `requests` + BeautifulSoup 配合 CSS Selector 解析；而 `/devices` 数据由 API 动态加载，必须先通过浏览器 Network 定位接口，再请求 JSON。两者分页方式也不同：HTML 分页依赖页面上的「下一页」链接循环跳转，API 分页则靠 `page` / `page_size` 参数并由 `total` 自动计算总页数。我通过总数与唯一 ID 校验、以及 Company 与 Device 的 `company_id` 数量一致性比对，确认采集结果完整且正确。
