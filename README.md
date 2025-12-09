# autodl-keeper
2024-06(🚧施工中) autodl自动续签 防止实例过期释放 

2024-07-20 已经测试一个月了 一切正常 可以正常续费

# 快速开始

## 克隆项目
```shell
git clone https://github.com/turbo-duck/autodl-keeper
cd autodl-keeper
````

![](./images/01.png)


## 新建配置
.env.template 为模板 你可以直接复制 或者 mv 修改名字
```shell
vim .env
```

写入内容为: 
- Authorization 为你的 Cookie
- MIN_DAY 为小于这个值则进行 开机-关机 的操作

```shell
Authorization=
MIN_DAY=7
```

## 获取Authorization
(这一块后续看是否可以自动化起来)
- vim 新建 .env
- 打开你的 AutoDL网页 F12
- 刷新一下 随便找一个接口
- 找到 Request Headers 部分
- 取出 Authorization 对应的值
- 取出的值 Copy 到 .env 的Authorization
- wq 退出vim

![](./images/02.png)

填写结果如下
![](./images/03.png)


## 启动方案1: 本地启动

```shell
pip3 install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```
或者
```shell
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```
![](./images/04.png)

启动服务
```shell
nohup python main.py &
```

## 查看日志
```shell
tail -f nohup.out
```

可以观察到，已经续费了。
![](./images/05.png)


## 启动方案2: 容器启动
你可以选择拉取现有镜像，或者自己打包。

**注意: 你需要查看 "新建配置" 的内容 需要配置一下 .env**

当前目录应该是这个样子:
```shell
.
├── Dockerfile
├── .env !注意这里是必须的!
├── .env.template
├── .git
├── .gitignore
├── LICENSE
├── main.py
├── nohup.out
├── README.md
└── requirements.txt
```

## 二选一: 拉取镜像
```shell
docker pull wdkang/autodl-keeper:v1.0
```
![](./images/08.png)

## 二选一: 打包镜像
```shell
docker build -t autodl-keeper .
```
![](./images/06.png)


## 启动镜像
```shell
docker run -d --env-file .env --name autodl-keeper autodl-keeper 
```
查看日志
```shell
docker logs -f autodl-keeper
```

![](./images/07.png)

## 启动方案3: GitHub Actions (推荐)

使用GitHub Actions可以免费运行定时任务，无需自己维护服务器。

### 配置步骤

1. **Fork 本项目**到你的GitHub账号

2. **配置Secrets**
   - 进入你的仓库，点击 `Settings` -> `Secrets and variables` -> `Actions`
   - 有两种配置方式：
   
   **方式1：使用 .env 文件（推荐）**
   - 点击 `New repository secret`，创建名为 `ENV_FILE` 的 secret
   - 将你本地的 `.env` 文件内容完整复制到 secret 的值中，例如：
     ```
     Authorization=your_authorization_token_here
     MIN_DAY=7
     ```
   
   **方式2：分别配置（备选）**
   - 点击 `New repository secret` 添加以下secrets:
     - `AUTODL_AUTHORIZATION`: 你的AutoDL Authorization token（获取方法见"获取Authorization"部分）
     - `MIN_DAY`: 最小天数阈值（可选，默认为7）

3. **启用Actions**
   - 进入 `Actions` 标签页
   - 如果提示需要启用，点击 `I understand my workflows, go ahead and enable them`

4. **手动触发测试（可选）**
   - 在 `Actions` 标签页，选择 `AutoDL Keeper` workflow
   - 点击 `Run workflow` 按钮手动触发一次测试

### 工作原理

- GitHub Actions会每天自动执行一次检查（UTC时间 0点，即北京时间早上8点）
- 你也可以在Actions页面手动触发执行
- 执行日志可以在Actions页面查看

### 优势

- ✅ 完全免费（GitHub Actions免费额度足够使用）
- ✅ 无需维护服务器
- ✅ 自动执行，无需担心服务器宕机
- ✅ 执行日志清晰可见

### 注意事项

- GitHub Actions使用UTC时间，cron表达式为 `0 0 * * *`（每天UTC 0点执行，即北京时间早上8点）
- 如果需要在特定时间执行，可以修改 `.github/workflows/autodl-keeper.yml` 中的cron表达式
- 免费账户每月有2000分钟的免费额度，每天执行一次完全足够使用
