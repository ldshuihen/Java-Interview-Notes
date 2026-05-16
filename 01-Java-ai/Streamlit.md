# Streamlit 

## Streamlit 详解

### 一、Streamlit 是什么

**Streamlit** 是一个**开源、免费**的 Python 框架，由前 Google 团队于 2018 年创建，2019 年正式发布，主打**用纯 Python 快速构建数据 Web 应用**，无需任何前端（HTML/CSS/JS）知识。
- 一句话定位：**把数据脚本变成可分享的网页，几分钟搞定，纯 Python**。
- 适用人群：数据科学家、机器学习工程师、分析师、Python 开发者。
- 官网：https://streamlit.io/



---

### 二、核心特点（为什么选它）
#### 1. 零前端门槛，纯 Python 开发
- 所有界面（按钮、滑块、图表、表格）都用 Python 函数生成，如 `st.button("点击")`、`st.slider("选择值", 0, 100)`。
- 无需写 HTML/CSS/JS，**会 Python 就能做网页应用**。

#### 2. 极简 API，几行代码出应用
```python
import streamlit as st
import pandas as pd

st.title("我的第一个 Streamlit 应用")  # 标题
st.write("Hello, Streamlit!")            # 文本
df = pd.DataFrame({"数值": [1, 2, 3]})   # 数据
st.dataframe(df)                          # 表格
st.line_chart(df)                         # 折线图
```
- 运行：`streamlit run app.py`，浏览器自动打开页面。

#### 3. 实时热重载，开发效率极高
- 保存代码 → 页面**自动刷新**，无需重启服务，类似 Jupyter 的开发体验。
- 交互触发：用户点击按钮、拖动滑块时，**整个脚本重新执行**，逻辑直观。

#### 4. 原生支持数据科学生态
- 无缝集成：**Pandas、NumPy、Matplotlib、Plotly、Scikit-learn、PyTorch、TensorFlow** 等。
- 一键显示：数据框、图表、图片、视频、地图、LaTeX 公式、进度条、文件上传。

#### 5. 内置丰富交互组件
- 输入类：滑块、下拉框、单选框、复选框、文本框、日期选择器、文件上传。
- 布局类：侧边栏（`st.sidebar`）、多列（`st.columns`）、折叠容器、标签页。
- 展示类：表格、数据卡片（`st.metric`）、图表、图片、音频、视频、地图。

#### 6. 部署简单，免费分享
- **Streamlit Community Cloud**：免费，关联 GitHub 仓库，一键部署公开应用。
- 其他方式：Docker、Heroku、AWS、本地服务器，支持跨平台（Windows/macOS/Linux）。

#### 7. 开源免费，社区活跃
- 许可证：**MIT**，商用免费。
- 社区：GitHub 星标超 30k，插件生态丰富（如 `streamlit-aggrid`、`streamlit-lottie`）。

---

### 三、安装与快速上手
#### 1. 安装
```bash
pip install streamlit
```

#### 2. 运行示例
```bash
streamlit hello
```
- 自动打开浏览器，预览官方示例应用。

#### 3. 第一个应用（app.py）
```python
import streamlit as st

# 页面配置
st.set_page_config(page_title="Demo", layout="wide")

# 标题与文本
st.title("Streamlit 快速演示")
st.subheader("纯 Python 构建网页应用")
st.write("这是普通文本，支持 **加粗**、*斜体*、[链接](https://streamlit.io)")

# 交互组件
name = st.text_input("请输入你的名字：")
if name:
    st.success(f"你好，{name}！")

age = st.slider("选择年龄", 0, 100, 25)
st.write(f"你选择的年龄是：{age}")

# 数据可视化
import pandas as pd
import numpy as np
df = pd.DataFrame({"X": np.random.randn(100), "Y": np.random.randn(100)})
st.line_chart(df, use_container_width=True)
```

#### 4. 启动应用
```bash
streamlit run app.py
```
- 浏览器访问：`http://localhost:8501`。

---

### 四、核心功能详解
#### 1. 页面与布局
- 标题/文本：`st.title()`、`st.header()`、`st.subheader()`、`st.write()`、`st.markdown()`。
- 侧边栏：`st.sidebar.title("侧边栏")`，组件加在侧边栏。
- 多列布局：
```python
col1, col2 = st.columns(2)
with col1:
    st.write("第一列")
with col2:
    st.write("第二列")
```

#### 2. 数据展示
- 表格：`st.dataframe(df)`（交互式，可排序/筛选）、`st.table(df)`（静态）。
- 数据卡片：`st.metric(label="销售额", value="¥12,345", delta="+6%")`。
- JSON/代码：`st.json(data)`、`st.code("print('hello')", language="python")`。

#### 3. 可视化图表
- 内置图表：`st.line_chart()`、`st.bar_chart()`、`st.area_chart()`、`st.scatter_chart()`。
- 第三方集成：
```python
import matplotlib.pyplot as plt
fig, ax = plt.subplots()
ax.plot([1, 2, 3])
st.pyplot(fig)  # Matplotlib

import plotly.express as px
fig = px.line(df, x="X", y="Y")
st.plotly_chart(fig)  # Plotly
```

#### 4. 交互组件
- 按钮：`if st.button("提交"): st.success("已提交")`。
- 选择框：`option = st.selectbox("选择", ["A", "B", "C"])`。
- 文件上传：`file = st.file_uploader("上传CSV", type="csv")`。
- 进度条：`progress = st.progress(0)`，动态更新进度。

#### 5. 缓存机制（性能优化）
- 避免重复计算（脚本每次交互都会重跑）：
```python
@st.cache_data  # 数据缓存（推荐）
def load_data():
    return pd.read_csv("data.csv")

@st.cache_resource  # 模型/大对象缓存
def load_model():
    return torch.load("model.pth")
```

---

### 五、优缺点分析
#### ✅ 优点
1. **学习成本极低**：只需 Python，无需前端。
2. **开发速度极快**：几行代码出应用，热重载高效。
3. **数据友好**：原生支持主流数据科学库，可视化便捷。
4. **部署分享简单**：Community Cloud 免费一键部署。
5. **社区活跃**：插件丰富，问题易解决。

#### ❌ 缺点
1. **前端定制弱**：页面样式、交互自由度不如 React/Vue。
2. **复杂应用受限**：不适合超大规模、高并发企业系统。
3. **状态管理简单**：依赖脚本重跑，复杂状态需手动维护。
4. **性能瓶颈**：大数据量/高交互时，响应速度可能下降。

---

### 六、典型应用场景
- **数据分析仪表盘**：快速构建业务数据可视化看板。
- **机器学习 Demo**：模型效果演示、参数调优界面。
- **AI 对话应用**：结合大模型（如 GPT、LLaMA）构建聊天界面。
- **数据工具内部化**：把 Python 脚本变成团队可用的网页工具。
- **教育演示**：算法、数据结构、统计分析的交互式教学工具。

---

### 七、部署方式
#### 1. Streamlit Community Cloud（推荐免费）
1. 代码推到 GitHub 仓库（公开）。
2. 访问 https://share.streamlit.io/，关联 GitHub 账号。
3. 选择仓库，一键部署，生成公开链接分享。

#### 2. 本地/服务器部署
```bash
# 后台运行
nohup streamlit run app.py --server.port=8501 &
```

#### 3. Docker 部署
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 8501
CMD ["streamlit", "run", "app.py", "--server.port=8501", "--server.address=0.0.0.0"]
```

---

### 八、总结
Streamlit 是**数据科学与 AI 领域的“神器”**，用**纯 Python** 即可快速构建**美观、交互、可分享**的 Web 应用，**零前端门槛、开发效率极高、部署简单**，特别适合数据分析、模型演示、内部工具等场景。虽然不适合超复杂企业级应用，但对**数据工作者**而言，是**投入产出比最高**的 Web 开发选择。

需要我把上述核心功能整理成一份可直接复制运行的**Streamlit 常用代码片段合集**吗？

## Streamlit 最全常用代码片段（直接复制运行）

### 一、基础页面配置

```python
import streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px

# 页面全局配置（必须放最顶部）
st.set_page_config(
    page_title="Streamlit工具箱",
    page_icon="🔥",
    layout="wide",  # wide 宽屏 / centered 居中
    initial_sidebar_state="expanded"
)
```

### 二、文本&标题组件

```python
# 各级标题
st.title("一级大标题")
st.header("二级标题")
st.subheader("三级小标题")

# 普通文本
st.write("普通正文内容，万能输出")

# 富文本Markdown
st.markdown("""
**加粗文字**  
*斜体文字*  
> 引用文本  
- 列表1  
- 列表2  
""")

# 代码块
st.code("print('Hello Streamlit')", language="python")

# 公式
st.latex(r"y = ax^2 + bx + c")
```

### 三、提示状态组件

```python
st.success("✅ 操作成功")
st.info("ℹ️ 温馨提示")
st.warning("⚠️ 警告提醒")
st.error("❌ 错误异常")
```

### 四、侧边栏所有组件

```python
# 侧边栏标题
st.sidebar.title("侧边栏控制面板")

# 侧边栏输入
side_text = st.sidebar.text_input("侧边栏输入框")
side_num = st.sidebar.number_input("侧边栏数字", min_value=0, max_value=100)
side_slide = st.sidebar.slider("侧边栏滑块", 0, 100, 50)
side_select = st.sidebar.selectbox("下拉选择", ["选项A","选项B","选项C"])
side_check = st.sidebar.checkbox("侧边栏复选框")
side_btn = st.sidebar.button("侧边栏按钮")
```

### 五、主流输入交互组件

```python
# 1.单行输入
name = st.text_input("请输入姓名")

# 2.多行文本
content = st.text_area("多行文本框")

# 3.数字输入
num = st.number_input("输入数字", value=10, step=1)

# 4.滑块
val = st.slider("滑动选择", min_value=1, max_value=50, value=20)

# 5.下拉单选
city = st.selectbox("选择城市", ["北京","上海","广州","深圳"])

# 6.多选框
hobby = st.multiselect("选择爱好", ["运动","阅读","游戏","音乐"])

# 7.单选按钮
sex = st.radio("选择性别", ["男","女"])

# 8.复选框
agree = st.checkbox("我已阅读协议")

# 9.日期选择
date = st.date_input("选择日期")

# 10.时间选择
time = st.time_input("选择时间")
```

### 六、按钮点击逻辑

```python
if st.button("点击执行"):
    st.write("按钮被点击啦！")
    st.balloons()  # 气球动画
    st.snow()       # 雪花动画
```

### 七、多列布局

```python
# 平分两列
col1, col2 = st.columns(2)
with col1:
    st.subheader("第一列")
    st.write("左侧内容")
with col2:
    st.subheader("第二列")
    st.write("右侧内容")

# 自定义比例三列
c1, c2, c3 = st.columns([1,2,1])
with c2:
    st.info("中间宽列")
```

### 八、折叠容器

```python
with st.expander("点击展开查看详情"):
    st.write("隐藏的详细内容")
    st.dataframe(pd.DataFrame({"数据":[1,2,3]}))
```

### 九、标签页切换

```python
tab1, tab2, tab3 = st.tabs(["数据预览","图表分析","模型结果"])
with tab1:
    st.write("数据页面")
with tab2:
    st.write("图表页面")
with tab3:
    st.write("模型页面")
```

### 十、数据表格展示

```python
# 构造测试数据
df = pd.DataFrame({
    "姓名":["张三","李四","王五"],
    "年龄":[22,25,28],
    "分数":[88,92,79]
})

# 交互式表格（可排序筛选）
st.dataframe(df, use_container_width=True)

# 静态表格
st.table(df)

# 指标卡片
st.metric(label="总人数", value="3人", delta="+1")
st.metric(label="平均分", value="86.3", delta="-2.1")
```

### 十一、内置极简图表

```python
st.line_chart(df, x="姓名", y="分数")
st.bar_chart(df, x="姓名", y="分数")
st.area_chart(df)
st.scatter_chart(df)
```

### 十二、Plotly高级图表

```python
fig = px.bar(df, x="姓名", y="分数", title="成绩柱状图")
st.plotly_chart(fig, use_container_width=True)
```

### 十三、文件上传功能

```python
# 上传单文件
upload_file = st.file_uploader("上传CSV/Excel文件", type=["csv","xlsx"])
if upload_file is not None:
    data = pd.read_csv(upload_file)
    st.dataframe(data)

# 多文件上传
files = st.file_uploader("多文件上传", accept_multiple_files=True)
```

### 十四、进度条&加载动画

```python
import time
progress_bar = st.progress(0)
for i in range(100):
    progress_bar.progress(i+1)
    time.sleep(0.02)
st.success("加载完成")

# 加载中状态
with st.spinner("正在疯狂运算中..."):
    time.sleep(2)
st.success("运算结束")
```

### 十五、缓存优化（核心提速）

```python
# 缓存数据（读取文件、查询数据）
@st.cache_data
def get_data():
    return pd.DataFrame(np.random.rand(50,3))

# 缓存模型（加载大模型、深度学习模型）
@st.cache_resource
def load_ai_model():
    return "大模型已加载"

data_cache = get_data()
st.dataframe(data_cache)
```

### 十六、下载文件功能

```python
csv_data = df.to_csv(index=False).encode("utf-8")
st.download_button(
    label="下载CSV数据",
    data=csv_data,
    file_name="数据表格.csv",
    mime="text/csv"
)
```

### 十七、图片/音频/视频

```python
# 本地图片
# st.image("test.jpg", caption="示例图片", use_column_width=True)

# 网络图片
st.image("https://streamlit.io/images/brand/streamlit-logo-secondary-colormark-darktext.png")
```

### 十八、一键运行命令

```bash
# 启动项目
streamlit run main.py

# 指定端口启动
streamlit run main.py --server.port=8888

# 外网可访问
streamlit run main.py --server.address=0.0.0.0
```

---
### 快速使用

1. 新建 `main.py`
2. 粘贴任意代码片段
3. 终端执行 `streamlit run main.py` 直接运行

需要我给你整合**一整页完整版聚合项目**，直接做成可上线的数据看板模板吗？

Streamlit 完整版聚合项目（通用万能模板）

**介绍：本项目整合 Streamlit 95% 常用功能 + 主流第三方库，一份代码包含所有常用案例，可直接作为个人永久模板。**

**使用方式：新建 main.py，全部粘贴，终端输入：streamlit run main.py**

## 完整可运行源码（直接复制）

```python
# ==============================================
# Streamlit 完整版聚合通用模板
# 包含：布局/组件/第三方图表/Excel/缓存/下载/AI/高级表格
# Author: 专属定制模板
# ==============================================
import streamlit as st
import pandas as pd
import numpy as np
import time
import plotly.express as px
import matplotlib.pyplot as plt
from st_aggrid import AgGrid
from streamlit_lottie import st_lottie
import requests

# ===================== 1.全局页面配置（必须放最顶部） =====================
st.set_page_config(
    page_title="全能综合平台",
    page_icon="🚀",
    layout="wide",
    initial_sidebar_state="expanded"
)

# ===================== 2.全局样式美化 =====================
def set_style():
    st.markdown("""
    <style>
    .main {background-color: #f7f8fa;}
    .block-container {padding-top: 2rem;}
    </style>
    """, unsafe_allow_html=True)
set_style()

# ===================== 3.侧边栏（菜单栏） =====================
with st.sidebar:
    st.title("📋 功能导航栏")
    menu = st.radio(
        "请选择功能模块",
        ["首页概览", "基础组件", "数据表格", "可视化图表", "文件上传下载", "AI简易对话", "动画美化"]
    )
    st.divider()
    st.success("✅ 完整版模板，可永久自用")
    st.sidebar.info("版本：V1.0 完整版")

# ===================== 4.构造通用测试数据 =====================
@st.cache_data
def create_data():
    df = pd.DataFrame({
        "姓名":["张三","李四","王五","赵六","钱七"],
        "部门":["技术部","运营部","市场部","技术部","运营部"],
        "薪资":[12000,9500,11000,13500,8800],
        "工龄":[3,2,4,5,2]
    })
    return df
df = create_data()

# ===================== 5.首页概览模块 =====================
if menu == "首页概览":
    st.title("🏠 Streamlit 全能聚合平台")
    st.markdown("""
    ### ✅ 本模板包含功能
    - 🔹 全套基础输入输出组件
    - 🔹 多列布局、标签页、折叠面板
    - 🔹 Matplotlib / Plotly 双图表
    - 🔹 高级AgGrid交互式表格
    - 🔹 Excel上传、数据下载
    - 🔹 缓存优化、进度条动画
    - 🔹 Lottie动画美化
    - 🔹 简易AI对话接口
    """)
    st.divider()

    # 指标卡片
    st.subheader("📊 核心数据指标")
    c1,c2,c3,c4 = st.columns(4)
    with c1:
        st.metric("总员工数","5人","+2")
    with c2:
        st.metric("平均薪资","10960元","+4.2%")
    with c3:
        st.metric("最高薪资","13500元","持平")
    with c4:
        st.metric("平均工龄","3.2年","-0.5")

# ===================== 6.基础组件模块 =====================
elif menu == "基础组件":
    st.title("🧩 全部基础组件演示")
    st.subheader("1.文本类组件")
    st.markdown("**加粗文字 | *斜体文字* | ~删除线~ | ==高亮==**")
    st.code("print('Hello Streamlit')",language="python")

    st.subheader("2.输入交互组件")
    col1,col2,col3 = st.columns(3)
    with col1:
        name = st.text_input("文本输入","请输入姓名")
        sex = st.radio("单选",["男","女"])
    with col2:
        num = st.number_input("数字输入",value=2026)
        slide = st.slider("滑块选择",0,100,50)
    with col3:
        select = st.selectbox("下拉选择",["A选项","B选项","C选项"])
        check = st.multiselect("多选",["吃饭","睡觉","敲代码"])

    st.subheader("3.提示状态组件")
    st.success("✅ 成功提示")
    st.info("ℹ️ 普通提示")
    st.warning("⚠️ 警告提示")
    st.error("❌ 错误提示")

    st.subheader("4.按钮特效")
    if st.button("点我触发动画"):
        st.balloons()
        st.snow()

# ===================== 7.数据表格模块 =====================
elif menu == "数据表格":
    st.title("📑 高级表格合集（第三方库）")
    tab1,tab2 = st.tabs(["原生表格","AgGrid高级表格"])
    with tab1:
        st.dataframe(df,use_container_width=True)
        st.table(df)
    with tab2:
        st.info("AgGrid：筛选、排序、复制、冻结、导出，企业级表格")
        AgGrid(df,height=400,theme="streamlit")

# ===================== 8.可视化图表模块 =====================
elif menu == "可视化图表":
    st.title("📈 第三方可视化图表大全")
    tab1,tab2,tab3 = st.tabs(["内置图表","Matplotlib","Plotly高级图表"])
    with tab1:
        st.bar_chart(df,x="姓名",y="薪资")
        st.line_chart(df,x="姓名",y="工龄")
    with tab2:
        fig,ax = plt.subplots(figsize=(6,3))
        ax.bar(df["姓名"],df["薪资"],color="#4ecdc4")
        st.pyplot(fig)
    with tab3:
        fig2 = px.pie(df,names="部门",values="薪资",title="部门薪资占比")
        st.plotly_chart(fig2,use_container_width=True)

# ===================== 9.文件上传下载模块 =====================
elif menu == "文件上传下载":
    st.title("📁 Excel 文件操作")
    upload = st.file_uploader("上传CSV/Excel",type=["csv","xlsx"])
    if upload:
        data_upload = pd.read_csv(upload)
        st.dataframe(data_upload)
        st.success("文件上传成功！")

    st.divider()
    st.subheader("数据下载功能")
    csv = df.to_csv(index=False).encode("utf-8-sig")
    st.download_button(
        label="📥 下载员工数据",
        data=csv,
        file_name="员工数据表.csv",
        mime="text/csv"
    )

# ===================== 10.AI简易对话模块 =====================
elif menu == "AI简易对话":
    st.title("🤖 简易AI交互模板（可直接对接大模型）")
    prompt = st.text_input("请输入你的问题")
    if prompt:
        with st.spinner("AI思考中..."):
            time.sleep(1.2)
        st.write(f"【AI回答】：收到你的问题：{prompt}，这是通用AI模板，可自行接入DeepSeek、GPT、通义千问。")

# ===================== 11.动画美化模块 =====================
elif menu == "动画美化":
    st.title("✨ Lottie动画美化（第三方美化库）")
    def load_lottie(url):
        return requests.get(url).json()
    lottie_ani = load_lottie("https://assets1.lottiefiles.com/packages/lf20_jcikwtux.json")
    st_lottie(lottie_ani,height=300)
    st.success("所有美化插件直接开箱即用")

# ==============================================
# 全套依赖安装命令
# pip install streamlit pandas numpy plotly matplotlib streamlit-aggrid streamlit-lottie openpyxl requests
# ==============================================
```

### 配套依赖安装命令（一次性全部装好）

复制下面一行，CMD终端执行：

```bash
pip install streamlit pandas numpy plotly matplotlib streamlit-aggrid streamlit-lottie openpyxl requests
```

### 项目特点（这就是你要的完整版聚合项目）

- **全部功能集成在一个py文件**，不用拆分，新手友好
- **包含所有第三方常用库**：Plotly、Matplotlib、AgGrid、Lottie、Requests
- **侧边栏菜单 + 标签页 + 多列布局**，企业级排版
- **自带缓存、美化样式、指标卡片、下载上传**
- **无冗余代码、注释清晰、可永久当做个人模板**