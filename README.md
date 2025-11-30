# 智慧停车场管理系统服务端（Hertz）

基于 CloudWeGo Hertz 的高性能 HTTP 服务端，提供智慧停车场的核心能力：车场/车位管理、车辆进出/计费、订单与支付、用户与权限、运维与监控。

## 技术栈
- Go 1.24（参考 `go.mod`）
- CloudWeGo Hertz（HTTP 框架）
- Thrift IDL（驱动路由与模型生成）
- MySQL、Redis（由 `driver/` 管理连接，规划中）

## 项目结构
- `main.go`：入口，初始化 Hertz 并注册路由
- `router_gen.go`：由 hz 根据 IDL 生成的路由注册代码
- `router.go`：自定义路由注册（如 `GET /ping`）
- `biz/handler/`：业务处理函数（控制器层）
- `biz/router/`：路由分组与中间件（含 IDL 生成的路由）
- `biz/model/`：模型层
  - `biz/model/smart_park/`：由 Thrift 生成的请求/响应结构体（DTO/契约模型），例如 `HelloReq`、`HelloResp`
  - `biz/model/do/`：数据库表对应的结构体（DO/实体模型），用于持久化映射（规划中）
- `driver/`：基础设施与连接管理（MySQL、Redis、配置与生命周期管理，规划中）
- `idl/`：Thrift 定义（例如 `smart_park.thrift`）
- `build.sh`、`script/bootstrap.sh`：构建与启动脚本

## 环境准备
### 安装 hz
```bash
go install github.com/cloudwego/hertz/cmd/hz@latest
hz -v
```
显示版本信息（如 `hz version v0.x.x`）表示安装成功。

## 快速开始
1. 安装依赖：
```bash
go mod tidy
```
2. 本地运行：
```bash
go run main.go
```
3. 基本连通性：
```bash
curl -s http://localhost:8888/ping
```
返回：
```json
{"message":"pong"}
```
4. 示例 API（IDL 生成）：
```bash
curl -s "http://localhost:8888/hello?name=Trae"
```
响应模型定义见 `biz/model/smart_park/smart_park.go`。

## 构建与发布
- 构建二进制：
```bash
./build.sh
```
产物将生成到 `output/`，包含 `output/bin/hertz_service` 与 `output/bootstrap.sh`。
- 启动产物：
```bash
./output/bootstrap.sh
```

## 配置约定（规划）
- 数据库：
  - `DB_DSN`，例如：`user:pass@tcp(127.0.0.1:3306)/smart_park?charset=utf8mb4&parseTime=True&loc=Local`
- Redis：
  - `REDIS_ADDR`，例如：`127.0.0.1:6379`
  - `REDIS_PASSWORD`（可选）、`REDIS_DB`（可选）

## 开发流程（IDL 驱动 + 模型分层）
1. 在 `idl/smart_park.thrift` 定义/更新接口契约
2. 生成/更新代码：
```bash
hz update -idl idl/smart_park.thrift
# 或
./update_idl.sh
```
3. 在 `biz/handler` 实现业务逻辑，并按需在 `biz/router/.../middleware.go` 增加中间件
4. 数据持久化：在 `biz/model/do` 中定义表映射结构体，在 `driver` 内实现 MySQL/Redis 访问（初始化连接、读写封装）

## 路由与中间件
- 自定义路由入口：`router.go`（`customizedRegister`）
- IDL 生成路由：`biz/router/smart_park/smart_park.go`
- 中间件示例：`biz/router/smart_park/middleware.go`

## 规划与待办
- 车场与车位管理：建模、增删改查、状态变更
- 入场/出场与计费：计费规则、通行记录
- 订单与支付：支付渠道对接、对账
- 用户与权限：角色/权限体系、登录鉴权
- 基础设施：`driver/` 完成 MySQL/Redis 连接管理与封装
- 运维能力：日志、指标、告警与健康检查

## 常用命令
```bash
hz -v                                   # 查看 hz 版本
hz update -idl idl/smart_park.thrift    # 更新 IDL 生成代码
./build.sh                              # 构建可执行文件
go run main.go                          # 运行本地开发
```

## 参考
- CloudWeGo Hertz: https://www.cloudwego.io/docs/hertz
- Thrift IDL: https://thrift.apache.org/
