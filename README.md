
## 👨‍💻 团队协作指南：数据集集成与贡献

### Part 1: 首次集成与运行环境设置

这是新成员第一次启动环境的流程。

#### 1.1 克隆仓库

团队成员需要在本地克隆 **Google 的工具仓库** 和 **我们的数据仓库**。

Bash

```
# 1. 克隆 OSS-Fuzz 核心工具
git clone https://github.com/google/oss-fuzz.git
cd oss-fuzz

# 2. 克隆团队数据集
# 注意：我们将所有项目都放在这个仓库里
git clone https://github.com/shenshiqiSSQ/fuzz_targets.git ~/fuzz_targets
```

#### 1.2 编译验证 (运行一个已知的项目)

团队成员需要修改 Dockerfile，告诉它从我们的 GitHub 仓库拉取数据。

Bash

```
# 在 ~/oss-fuzz 目录下操作

# 1. 修正 libspng 的 Dockerfile (指向我们的团队仓库)
# (团队领导需确保 libspng/Dockerfile 里的 git clone 地址已改为 shenshiqiSSQ 的仓库)

# 2. 编译并运行验证 (测试 libspng 的 Double Free 漏洞)
python3 infra/helper.py build_fuzzers --sanitizer address libspng ~/fuzz_targets/libspng
python3 infra/helper.py run_fuzzer libspng libspng_read_fuzzer --max_time=30
```

_（如果能看到红色的 ASAN 崩溃报告，则环境集成成功。）_

---

### Part 2: 如何修改已经存在的项目内容

当你需要更新 `miniz` 里的漏洞（例如发现之前的 Double Free 不够好）时，你需要先同步主仓库，再进行修改。

#### 2.1 同步与编辑 (Pull & Edit)

Bash

```
# 1. 切换到数据仓库根目录
cd ~/fuzz_targets

# 2. 从 GitHub 拉取最新的修改 (避免覆盖队友的代码)
git pull origin main

# 3. 进入 miniz 目录开始修改代码
cd miniz
nano miniz.c # 修正或替换漏洞代码
```

#### 2.2 验证与推送 (Verify & Push)

Bash

```
# 1. 编译验证 (在 oss-fuzz 目录下，验证修改是否能通过编译和崩溃测试)
cd ~/oss-fuzz
python3 infra/helper.py build_fuzzers --sanitizer address miniz ~/fuzz_targets/miniz

# 2. 提交到主仓库
cd ~/fuzz_targets
git add miniz/
git commit -m "fix: Updated miniz UAF logic and verified compilation."
git push origin main
```

---

### Part 3: 如何添加新的漏洞项目 (项目扩展)

当你决定将一个全新的项目（例如 `curl`）添加到数据集中时，这是完整的流程。

#### 3.1 搭建本地结构

Bash

```
# 1. 下载新项目源码
cd ~/fuzz_targets
git clone <curl 官方仓库 URL> curl # 下载 curl 源码

# 2. 创建 Fuzz Harness (手动创建测试入口文件)
# cd curl
# nano curl_fuzzer.c # 编写 LLVMFuzzerTestOneInput 函数

# 3. 注入漏洞
# nano curl/lib/http.c # 植入 5 个漏洞
```

#### 3.2 配置 OSS-Fuzz 配方

这是最关键的一步，你需要为 **curl** 创建它的“编译说明书”。

Bash

```
# 1. 创建项目目录 (在 oss-fuzz/projects 下)
cd ~/oss-fuzz/projects
mkdir curl

# 2. 创建 Dockerfile (安装依赖，如 libssl-dev)
nano curl/Dockerfile 
# (粘贴依赖安装和拉取团队代码的配置)

# 3. 创建 Build.sh (配置 Autotools/编译 libcurl.a)
nano curl/build.sh
# (粘贴 autogen/configure/make 流程)

# --- EXECUTION ---

# 4. 编译并运行验证 (测试 CURL 项目)
# 注意：使用 curl 作为项目名，并挂载 ~/fuzz_targets/curl
cd ~/oss-fuzz
python3 infra/helper.py build_fuzzers --sanitizer address curl ~/fuzz_targets/curl

# 5. 运行 Fuzzer (验证是否有漏洞)
python3 infra/helper.py run_fuzzer curl curl_fuzzer -- -max_total_time=30

# 6. 推送 CURL 文件夹和更新后的配方
cd ~/fuzz_targets
git add curl  # 添加新项目文件夹
cd ~/oss-fuzz
git add projects/curl/ # 添加新的配方文件
git commit -m "feat: Added Project CURL (Build config and code)."
git push origin main
```
