# 使用 AutoScraper 进行网页爬虫

[![推广](https://github.com/bright-cn/Awesome-Web-Scraping/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner%20CN.png)](https://www.bright.cn/)

本指南介绍如何在 Python 中使用 AutoScraper 进行网页爬虫：

- [什么是 AutoScraper？](#什么是-autoscraper)
- [设置项目](#设置项目)
- [选择目标网站](#选择目标网站)
- [使用 AutoScraper 抓取简单数据](#使用-autoscraper-抓取简单数据)
- [从设计复杂的网站中提取数据](#从设计复杂的网站中提取数据)
- [AutoScraper 的常见挑战](#autoscraper-的常见挑战)

## 什么是 AutoScraper？

[AutoScraper](https://github.com/alirezamika/autoscraper) 是一个 Python 库，它能根据示例查询自动识别并提取网站数据，从而简化网页爬虫流程，无需手动检查 HTML。只需极少的设置，它就能高效处理动态网站。在[此处](https://www.bright.cn/blog/how-tos/scrape-dynamic-websites-python)详细了解如何抓取动态网站。

## 设置项目

安装 Python 3 或更高版本后，创建一个项目目录并进入该目录：

```bash
mkdir auto-scrape && cd auto-scrape
```

创建虚拟环境：

```bash
python -m venv env
```

然后激活它。在 Linux 和 macOS 上：

```bash
source env/bin/activate
```

在 Windows 上：

```powershell
venv\Scripts\activate
```

安装 `autoscraper` 和 `pandas` 库：

```bash
pip install autoscraper pandas
```

## 选择目标网站

抓取公共网站时，请务必查看该网站的服务条款 (ToS) 或 `robots.txt` 文件，以确保网站允许抓取。

本指南假设要从 [Scrape This Site 的“世界各国：简单示例”页面](https://www.scrapethissite.com/pages/simple/)抓取数据，该页面是一个专为测试抓取工具而设计、适合初学者的沙盒。

该页面结构简单，非常适合作为学习数据提取基础知识的起点。掌握基本概念后，我们将继续学习更复杂的示例：[冰球队：表单、搜索和分页页面](https://www.scrapethissite.com/pages/forms/)。

## 使用 AutoScraper 抓取简单数据

使用以下脚本抓取国家/地区列表及其首都、人口和面积：

```python
# 1. Import dependencies
from autoscraper import AutoScraper
import pandas as pd

# 2. Define the URL of the site to be scraped
url = "https://www.scrapethissite.com/pages/simple/"

# 3. Instantiate the AutoScraper
scraper = AutoScraper()

# 4. Define the wanted list by using an example from the web page
# This list should contain some text or values that you want to scrape
wanted_list = ["Andorra", "Andorra la Vella", "84000", "468.0"]

# 5. Build the scraper based on the wanted list and URL
scraper.build(url, wanted_list)

# 6. Get the results for all the elements matched
results = scraper.get_result_similar(url, grouped=True)

# 7. Display the keys and sample data to understand the structure
print("Keys found by the scraper:", results.keys())

# 8. Assign columns based on scraper keys and expected order of data
columns = ["Country Name", "Capital", "Area (sq km)", "Population"]

# 9. Create a DataFrame with the extracted data
data = {columns[i]: results[list(results.keys())[i]] for i in range(len(columns))}
df = pd.DataFrame(data)

# 10. Save the DataFrame to a CSV file
csv_filename = 'countries_data.csv'
df.to_csv(csv_filename, index=False)

print(f"Data has been successfully saved to {csv_filename}")
```

上述脚本首先导入必要的库：AutoScraper 和 pandas。接下来定义目标网站的 URL，然后创建一个 AutoScraper 实例。

这正是 AutoScraper 的突出之处：传统爬虫工具需要 XPath 或 CSS 选择器等明确指令，而 AutoScraper 允许你提供示例数据。你无需指定元素在页面上的位置，只需列出几个想要提取的示例值。这个名为 `wanted_list` 的列表包含具有代表性的数据点，例如国家/地区名称、首都、人口和面积。

设置 `wanted_list` 后，爬虫工具会使用所提供的 URL 和示例数据进行构建。随后，AutoScraper 会分析网页、识别模式并生成抓取规则。这些规则使爬虫能够从相同或其他网页中识别并提取类似数据。

在脚本后面的部分，会调用 `get_result_similar` 方法来检索符合 AutoScraper 已学习模式的所有数据。为了更好地了解抓取工具如何解释数据结构，脚本会输出与提取数据关联的规则 ID。输出应类似于：

```
Keys found by the scraper: dict_keys(['rule_4y6n', 'rule_gghn', 'rule_a6r9', 'rule_os29'])
```

注释八和注释九定义列名，并将提取的数据整理为 pandas DataFrame。注释十随后将这些数据保存为 CSV 文件。

运行脚本 (`python script.py`) 后，项目目录中会出现一个 `countries_data.csv` 文件，其内容类似于：

```csv
Country Name,Capital,Area (sq km),Population
Andorra,Andorra la Vella,84000,468.0
United Arab Emirates,Abu Dhabi,4975593,82880.0
...246 collapsed rows
Zambia,Lusaka,13460305,752614.0
Zimbabwe,Harare,11651858,390580.0
```

## 从设计复杂的网站中提取数据

上述方法可能难以处理[冰球队页面](https://www.scrapethissite.com/pages/forms/)这类复杂网站，因为该页面包含一个具有大量相似值的表格。尝试使用相同方法提取球队名称、年份、胜场数和其他字段，以了解其中的挑战。

幸运的是，AutoScraper 支持在构建步骤中通过删减不必要的规则来优化其模型。你可以按以下方式操作：

```python
from autoscraper import AutoScraper
import pandas as pd

# Define the URL of the site to be scraped
url = "https://www.scrapethissite.com/pages/forms/"

def setup_model():

    # Instantiate the AutoScraper
    scraper = AutoScraper()

    # Define the wanted list by using an example from the web page
    # This list should contain some text or values that you want to scrape
    wanted_list = ["Boston Bruins", "1990", "44", "24", "0.55", "299", "264", "35"]

    # Build the scraper based on the wanted list and URL
    scraper.build(url, wanted_list)

    # Get the results for all the elements matched
    results = scraper.get_result_similar(url, grouped=True)

    # Display the data to understand the structure
    print(results)

    # Save the model
    scraper.save("teams_model.json")

def prune_rules():
    # Create an instance of Autoscraper
    scraper = AutoScraper()
    
    # Load the model saved earlier
    scraper.load("teams_model.json")

    # Update the model to only keep necessary rules
    scraper.keep_rules(['rule_hjk5', 'rule_9sty', 'rule_2hml', 'rule_3qvv', 'rule_e8x1', 'rule_mhl4', 'rule_h090', 'rule_xg34'])

    # Save the updated model again
    scraper.save("teams_model.json")
    
def load_and_run_model():
    # Create an instance of Autoscraper
    scraper = AutoScraper()
    
    # Load the model saved earlier
    scraper.load("teams_model.json")

    # Get the results for all the elements matched
    results = scraper.get_result_similar(url, grouped=True)

    # Assign columns based on scraper keys and expected order of data
    columns = ["Team Name", "Year", "Wins", "Losses", "Win %", "Goals For (GF)", "Goals Against (GA)", "+/-"]

    # Create a DataFrame with the extracted data
    data = {columns[i]: results[list(results.keys())[i]] for i in range(len(columns))}
    df = pd.DataFrame(data)

    # Save the DataFrame to a CSV file
    csv_filename = 'teams_data.csv'
    df.to_csv(csv_filename, index=False)

    print(f"Data has been successfully saved to {csv_filename}")

# setup_model()
# prune_rules()
# load_and_run_model()
```

该脚本定义了三个方法：`setup_model`、`prune_rules` 和 `load_and_run_model`。

`setup_model` 方法会创建一个爬虫实例、定义 `wanted_list`、构建爬虫、从目标 URL 抓取数据、输出采集到的规则 ID，并将模型以 `teams_model.json` 的名称保存到项目目录中。

要运行该脚本，请取消脚本中 `# setup_model()` 行的注释，保存文件（例如 `script.py`），然后使用 `python script.py` 运行它。

输出如下所示：

```
{'rule_hjk5': ['Boston Bruins', 'Buffalo Sabres', 'Calgary Flames', 'Chicago Blackhawks', 'Detroit Red Wings', 'Edmonton Oilers', 'Hartford Whalers', 'Los Angeles Kings', 'Minnesota North Stars', 'Montreal Canadiens', 'New Jersey Devils', 'New York Islanders', 'New York Rangers', 'Philadelphia Flyers', 'Pittsburgh Penguins', 'Quebec Nordiques', 'St. Louis Blues', 'Toronto Maple Leafs', 'Vancouver Canucks', 'Washington Capitals', 'Winnipeg Jets', 'Boston Bruins', 'Buffalo Sabres', 'Calgary Flames', 'Chicago Blackhawks'], 'rule_uuj6': ['Boston Bruins', 'Buffalo Sabres', 'Calgary Flames', 'Chicago Blackhawks', 'Detroit Red Wings', 'Edmonton Oilers', 'Hartford Whalers', 'Los Angeles Kings', 'Minnesota North Stars', 'Montreal Canadiens', 'New Jersey Devils', 'New York Islanders', 'New York Rangers', 'Philadelphia Flyers', 'Pittsburgh Penguins', 'Quebec Nordiques', 'St. Louis Blues', 'Toronto Maple Leafs', 'Vancouver Canucks', 'Washington Capitals', 'Winnipeg Jets', 'Boston Bruins', 'Buffalo Sabres', 'Calgary Flames', 'Chicago Blackhawks'], 'rule_9sty': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_9nie': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_41rr': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_ufil': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_ere2': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_w0vo': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_rba5': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_rmae': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_ccvi': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_3c34': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_4j80': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_oc36': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_93k1': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_d31n': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_ghh5': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_5rne': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_4p78': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_qr7s': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_60nk': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_wcj7': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_0x7y': ['1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1990', '1991', '1991', '1991', '1991'], 'rule_2hml': ['44', '31', '46', '49', '34', '37', '31', '46', '27', '39', '32', '25', '36', '33', '41', '16', '47', '23', '28', '37', '26', '36', '31', '31', '36'], 'rule_swtb': ['24'], 'rule_e8x1': ['0.55', '14', '0.575', '0.613', '-25', '0', '-38', '0.575', '-10', '24', '8', '-67', '32', '-15', '0.512', '-118', '0.588', '-77', '-72', '0', '-28', '-5', '-10', '-9', '21'], 'rule_3qvv': ['24', '30', '26', '23', '38', '37', '38', '24', '39', '30', '33', '45', '31', '37', '33', '50', '22', '46', '43', '36', '43', '32', '37', '37', '29'], 'rule_n07w': ['24', '30', '26', '23', '38', '37', '38', '24', '39', '30', '33', '45', '31', '37', '33', '50', '22', '46', '43', '36', '43', '32', '37', '37', '29'], 'rule_qmem': ['0.55', '0.388', '0.575', '0.613', '0.425', '0.463', '0.388', '0.575', '0.338', '0.487', '0.4', '0.312', '0.45', '0.412', '0.512', '0.2', '0.588', '0.287', '0.35', '0.463', '0.325', '0.45', '0.388', '0.388', '0.45'], 'rule_b9gx': ['264', '278', '263', '211', '298', '272', '276', '254', '266', '249', '264', '290', '265', '267', '305', '354', '250', '318', '315', '258', '288', '275', '299', '305', '236'], 'rule_mhl4': ['299', '292', '344', '284', '273', '272', '238', '340', '256', '273', '272', '223', '297', '252', '342', '236', '310', '241', '243', '258', '260', '270', '289', '296', '257'], 'rule_24nt': ['264', '278', '263', '211', '298', '272', '276', '254', '266', '249', '264', '290', '265', '267', '305', '354', '250', '318', '315', '258', '288', '275', '299', '305', '236'], 'rule_h090': ['264', '278', '263', '211', '298', '272', '276', '254', '266', '249', '264', '290', '265', '267', '305', '354', '250', '318', '315', '258', '288', '275', '299', '305', '236'], 'rule_xg34': ['35', '14', '81', '73', '-25', '0', '-38', '86', '-10', '24', '8', '-67', '32', '-15', '37', '-118', '60', '-77', '-72', '0', '-28', '-5', '-10', '-9', '21']}
```

此输出展示了 AutoScraper 在对目标网站执行 `get_result_similar` 调用时采集的数据。其中包含重复项，因为 AutoScraper 会尝试推断关系并将数据归入不同规则。正确分组后，就能轻松地从类似网站中提取数据。

但是，由于该网站包含大量数字，AutoScraper 难以正确处理，从而产生许多错误关联以及包含重复项的大型数据集。

要继续操作，请分析数据并选择包含正确数据的规则（即数据以正确顺序与某一列的数据匹配）。在本例中，以下规则包含正确的数据（通过确认每条规则均包含 25 个数据点来确定，这与目标页面的行数一致）：

```
['rule_hjk5', 'rule_9sty', 'rule_2hml', 'rule_3qvv', 'rule_e8x1', 'rule_mhl4', 'rule_h090', 'rule_xg34']
```

更新 `prune_rules` 方法。然后注释掉脚本中的 `setup_model()` 行，并取消 `prune_rules()` 行的注释。运行脚本时，它会从 `teams_model.json` 加载模型、删除不必要的规则并保存更新后的模型。你可以检查 `teams_model.json` 的内容，以确认其中存储的规则。

接下来，要运行 `load_and_run_model` 方法，请注释掉 `prune_rules` 行，取消 `load_and_run_model` 行的注释，然后重新运行脚本。这会提取正确的数据并将其保存到项目目录中的 `teams_data.csv`，同时输出以下内容：

```
Data has been successfully saved to teams_data.csv
```

成功运行后，`teams_data.csv` 文件如下所示：

```csv
Team Name,Year,Wins,Losses,Win %,Goals For (GF),Goals Against (GA),+/-
Boston Bruins,1990,44,0.55,24,299,264,35
Buffalo Sabres,1990,31,14,30,292,278,14
...21 more rows
Calgary Flames,1991,31,-9,37,296,305,-9
Chicago Blackhawks,1991,36,21,29,257,236,21
```

本文开发的所有代码都可以在[此 GitHub 仓库](https://github.com/krharsh17/auto-scrape)中找到。

## AutoScraper 的常见挑战

AutoScraper 非常适合处理数据集较小且数据点差异明显的简单用例。但对于抓取表格等更复杂的场景，使用起来可能较为繁琐。它也不支持 JavaScript 渲染，因此你需要将其与 [Splash](https://pypi.org/project/splash/)、Selenium 或 Puppeteer 等工具集成。

如果遇到 IP 被封锁或需要自定义标头等问题，AutoScraper 允许你为其 requests 模块指定额外的请求参数，如下所示：

```python
# build the scraper on an initial URL
scraper.build(
    url,
    wanted_list=wanted_list,
    request_args=dict(proxies=proxies) # this is where you can pass in a list of proxies or customer headers
)
```

例如，你可以按以下方式设置自定义 User-Agent 和代理，以使用 AutoScraper 进行抓取：

```python
request_args = { 
  "headers: {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_5) AppleWebKit/537.36 \
            (KHTML, like Gecko) Chrome/84.0.4147.135 Safari/537.36"  # You can customize this value with your desired user agent. This value is the default used by Autoscraper.
  },
  "proxies": {
    "http": "http://user:pass@10.10.1.10:3128/" # Example proxy to show you how to use the proxy port, host, username, and password values
  }
}
# build the scraper on an initial URL
scraper.build(
    url,
    wanted_list=wanted_list,
    request_args=request_args
)
```

AutoScraper 使用 Python 的 requests 库，但不支持速率限制。要处理速率限制，你可以手动实现节流，或使用 [`ratelimit`](https://pypi.org/project/ratelimit/) 库等预构建解决方案。

由于 AutoScraper 只能处理非动态网站，因此无法处理受 CAPTCHA 保护的网站。对于这些情况，可以考虑使用更强大的解决方案，例如 [Bright Data 网页爬虫工具 API](https://www.bright.cn/products/web-scraper)，它支持从 LinkedIn、Amazon 和 Zillow 等网站提取结构化数据。

立即注册并开始探索 Bright Data 的产品，包括免费试用！
