# 项目移交说明书：TradeCheck (跨境券商交易网络合规自测工具)

## 1. 项目背景与业务痛点 (Background & Core Business Logic)
随着跨境券商（富途牛牛、老虎证券、长桥证券等）对中国内地 IP 的交易风控收紧，内地交易者在普通代理模式下经常遭遇间歇性风控拦截。

### 核心业务痛点（产品经理精细化定义）
券商的合规风控采取**“最小化精准拦截”**策略：**看盘（大盘行情）不限、平仓（卖出动作）不限，唯独精准锁死内地 IP 的【买入下单】动作**。
* **技术穿透风险：** 券商 App（如富途）底层连接极其狡猾。点击“买入”时，App 会强制唤醒底层高优先级安全通道。如果软路由（Mihomo/OpenClash）的 TUN 虚拟网卡未完全锁死 53 端口或存在硬编码 IP 直连，App 就会绕过系统 DNS 发生 **DNS 泄露** 或 **IP 抢跑**，暴露出国内真实出口。
* **项目目标：** 打造一个轻量、极简、现代暗黑科技风的单页自适应工具网页。不仅提供常规 IP 分流对比，更针对券商“大盘/买入/卖出”三大行为链路进行**分项独立探测**，让用户直观看到自己的分流规则是否 100% 免疫买入风控。

---

## 2. 产品结构与原型设计 (Product Architecture)
网站整体风格对齐主流高级 IP 探测网站（类似 `ip.skk.moe` 的极客流），全面自适应手机端与电脑端。

### Tab 1: 本机 IP 查询（前端静态排版已完全锁定，请勿改动）
* **核心信息：** 展示 IPv4/IPv6、地理位置（Geofencing 基准）、ISP 运营商、ASN 信息。
* **分流对比：** 对比国内（ip138、IP.cn）与国际（Cloudflare、IPInfo）探测出的不同 IP，让用户直观看到软路由分流是否生效。
* **常用连通性：** 微信、淘宝（国内直连）与 GitHub、YouTube（境外代理）的实时延迟与丢包点阵。

### Tab 2: 券商探测（精细化行为诊断面板 - 核心重构区）
不再提供含糊的“券商整体状态”，而是彻底重构为**全行为链路诊断网格**：
1. **全局检测基准 IP：** 页面顶部显著展示未发出券商请求前，用户当前的默认国际出口 IP 与国家（作为对照组）。
2. **券商综合诊断状态栏：** 每一个券商卡片头部，根据下方子链路结果，动态组合并高亮展示结论（例如：`综合诊断：大盘正常 | 买入异常 | 卖出正常`）。
3. **分项链路探测组（网格三列平铺，手机端自动瀑布流）：**
   * **📈 访问大盘的 IP 位置：** 模拟 App 调取 CDN 行情接口。
   * **🛒 访问交易买入的 IP 位置：** 模拟核心下单风控接口（此处最容易发生国内 IP 抢跑或 DNS 泄露，若失败触发红色高危警告）。
   * **💰 访问交易卖出的 IP 位置：** 模拟资产清仓/平仓接口。

---

## 3. 当前项目资产 (Project Assets)
目前已完成全自适应前端单文件 `index.html` 的搭建（采用 Tailwind CSS 构建样式，包含响应式布局、Tab 切换交互逻辑、以及完全锁定形态的 UI 框架）。

<details>
<summary>点击展开查看最新的 index.html 源码</summary>

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TradeCheck - 跨境交易网络合规自测工具</title>
    <script src="[https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4](https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4)"></script>
    <style>body { background-color: #0b0c0e; color: #e2e8f0; }</style>
</head>
<body class="font-sans antialiased min-h-screen flex flex-col">
    <header class="border-b border-gray-800 bg-[#121418] sticky top-0 z-50 px-4 py-3">
        <div class="max-w-6xl mx-auto flex items-center justify-between">
            <nav class="flex space-x-2 bg-[#1a1d24] p-1 rounded-xl w-full sm:w-auto">
                <button id="tab-ip" onclick="switchTab('ip')" class="flex-1 sm:flex-none text-center px-5 py-2 rounded-lg text-sm font-medium transition-all bg-gray-800 text-white shadow">本机 IP</button>
                <button id="tab-broker" onclick="switchTab('broker')" class="flex-1 sm:flex-none text-center px-5 py-2 rounded-lg text-sm font-medium transition-all text-gray-400 hover:text-gray-200">券商探测</button>
            </nav>
        </div>
    </header>

    <main class="flex-1 max-w-6xl w-full mx-auto p-4 sm:p-6">
        <section id="content-ip" class="space-y-6">
            <div>
                <h1 class="text-2xl font-bold tracking-tight text-white">本机 IP 查询</h1>
                <p class="text-sm text-gray-400 mt-1">查看你当前网络的 IP 地址、位置信息和常用网站连通性</p>
            </div>
            <div class="grid grid-cols-1 lg:grid-cols-12 gap-4">
                <div class="lg:col-span-5 bg-[#121418] border border-gray-800 rounded-2xl p-5 relative overflow-hidden">
                    <div class="flex justify-between items-start">
                        <div class="space-y-4 w-full">
                            <div><label class="text-xs font-semibold text-gray-500">IPv4</label><p class="text-lg font-mono font-bold text-white">188.253.4.150</p></div>
                            <div><label class="text-xs font-semibold text-gray-500">Location</label><p class="text-sm font-medium text-gray-200">Singapore</p></div>
                            <div><label class="text-xs font-semibold text-gray-500">ISP</label><p class="text-sm font-medium text-gray-200">Akari Networks</p></div>
                        </div>
                        <span class="text-3xl">🇸🇬</span>
                    </div>
                </div>
                <div class="lg:col-span-7 bg-[#121418] border border-gray-800 rounded-2xl p-5 space-y-4">
                    <div class="flex justify-between items-center pb-3 border-b border-gray-800/60">
                        <div><h4 class="text-sm font-bold text-gray-200">ip138.com</h4><p class="text-xs font-mono text-gray-400">IP: 114.243.105.245</p></div>
                        <span class="px-2 py-0.5 text-[10px] bg-blue-900/40 text-blue-400 border border-blue-800 rounded-md">国内</span>
                    </div>
                    <div class="flex justify-between items-center">
                        <div><h4 class="text-sm font-bold text-gray-200">Cloudflare</h4><p class="text-xs font-mono text-gray-400">IP: 188.253.4.150</p></div>
                        <span class="px-2 py-0.5 text-[10px] bg-emerald-900/40 text-emerald-400 border border-emerald-800 rounded-md">国际</span>
                    </div>
                </div>
            </div>
            <div class="bg-[#121418] border border-gray-800 rounded-2xl p-5">
                <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6">
                    </div>
            </div>
        </section>

        <section id="content-broker" class="hidden space-y-6">
            <div class="bg-[#121418] border border-gray-800 rounded-2xl p-5 flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
                <div>
                    <h1 class="text-2xl font-bold tracking-tight text-white">跨境券商合规探测</h1>
                    <p class="text-sm text-gray-400 mt-1">核心风控精准定位：由于当前规则主要限制<b>内地 IP 执行买入动作</b>，本工具对各项核心行为进行链路细分诊断。</p>
                </div>
                <div class="bg-[#1a1d24] border border-gray-700 px-4 py-2.5 rounded-xl flex items-center space-x-3 w-full sm:w-auto">
                    <div class="w-2.5 h-2.5 bg-emerald-400 rounded-full animate-pulse"></div>
                    <div>
                        <p class="text-[10px] uppercase font-bold text-gray-500 tracking-wider">当前检测基准 IP</p>
                        <p class="text-sm font-mono font-bold text-white">188.253.4.150 <span class="text-xs text-emerald-400 ml-1">(🇸🇬 新加坡)</span></p>
                    </div>
                </div>
            </div>

            <div class="space-y-4">
                <div class="bg-[#121418] border border-gray-800 rounded-2xl p-5 space-y-4">
                    <div class="flex flex-col md:flex-row md:items-center justify-between pb-4 border-b border-gray-800 gap-2">
                        <div class="flex items-center space-x-3"><span class="text-xl">📊</span><h3 class="text-base font-bold text-white">富途证券 (Futu Futu)</h3></div>
                        <div class="flex items-center space-x-2 bg-orange-950/40 border border-orange-900/60 px-3 py-1.5 rounded-xl">
                            <span class="text-xs text-gray-400">综合诊断：</span>
                            <span class="text-xs font-bold text-emerald-400">大盘正常</span><span class="text-xs text-gray-500">|</span>
                            <span class="text-xs font-bold text-red-400">买入异常</span><span class="text-xs text-gray-500">|</span>
                            <span class="text-xs font-bold text-emerald-400">卖出正常</span>
                        </div>
                    </div>
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-3">
                        <div class="bg-[#1a1d24] border border-gray-800/80 rounded-xl p-3.5 space-y-2">
                            <span class="text-xs text-gray-400 font-medium">📈 访问大盘的 IP 位置</span>
                            <p class="text-sm font-mono font-bold text-white">188.253.4.150</p>
                        </div>
                        <div class="bg-red-950/10 border border-red-900/40 rounded-xl p-3.5 space-y-2">
                            <span class="text-xs text-red-400 font-bold">🛒 访问交易买入的 IP 位置</span>
                            <p class="text-sm font-mono font-bold text-red-400">114.243.105.245</p>
                        </div>
                        <div class="bg-[#1a1d24] border border-gray-800/80 rounded-xl p-3.5 space-y-2">
                            <span class="text-xs text-gray-400 font-medium">💰 访问交易卖出的 IP 位置</span>
                            <p class="text-sm font-mono font-bold text-white">188.253.4.150</p>
                        </div>
                    </div>
                </div>
                </div>
        </section>
    </main>

    <script>
        function switchTab(tab) {
            // 本地 Tab 切换逻辑（已实现）
        }
    </script>
</body>
</html>