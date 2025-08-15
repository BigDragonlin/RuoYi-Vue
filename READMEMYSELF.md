## IntelliJ IDEA 远程调试后端（Docker 容器中的 Spring Boot）

本项目已在 `docker-compose.yml` 的 `backend` 服务中开启 JDWP 远程调试：

- JVM 参数：`-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005`
- 端口映射：`5005:5005`

因此只需在 IDEA 中新建一个远程调试配置并附加到 `localhost:5005` 即可。

### 一、（可选）启动或重建后端容器

确保容器处于运行状态；若刚修改过 `docker-compose.yml`，重建一次使调试参数生效：

```bash
docker compose up -d --force-recreate --no-deps backend
docker compose logs -f backend
```

日志中应能看到类似：`Listening for transport dt_socket at address: 5005`。

### 二、在 IntelliJ IDEA 新建“远程调试”配置

1) 打开 Run → Edit Configurations…（或右上角配置下拉 → Edit Configurations…）
2) 点击左上角 “+” 新建配置，选择：Remote JVM Debug（旧版本叫 Remote/Attach to remote JVM）
3) 关键参数：
   - Host：`localhost`
   - Port：`5005`
   - 其他保持默认（Debugger mode: Attach to remote JVM / Socket）
4) 应用并保存，然后点击 Debug ▶️ 进入调试附加状态。

提示：若你希望应用在启动时等待调试器连接，可把 compose 中的 `suspend=n` 改为 `suspend=y`（默认无需改）。

### 三、设置与查看断点

- 在编辑器左侧行号区域单击添加断点（红点）。
- 右键断点可配置条件（Condition）、命中次数（Hit count）、只打印日志不暂停（Log evaluated expression）等。
- 查看/管理所有断点：
  - 菜单：Run → View Breakpoints…
  - 快捷键：Ctrl+Shift+F8（Windows/Linux）
- 调试窗口（Alt+5）中可查看：当前命中的断点、调用栈（Frames）、变量（Variables）、监视（Watches）。
- 确认未开启 “Mute Breakpoints”（调试窗口的小静音图标应未按下）。

断点颜色含义：
- 红色实心：断点已绑定，类已加载，会生效。
- 灰色空心：类尚未加载或源码未匹配，触发一次对应代码路径后会变为实心；若一直灰色，请参见下方排查。

### 四、验证断点是否命中

1) 在常用的 `Controller` 或 `Service` 方法上设断点。
2) 通过前端页面操作或直接访问接口触发该方法，例如：

```bash
curl http://localhost:8080/
```

3) 命中后 IDEA 会自动切到 Debug 窗口并在断点行高亮。

### 五、常见问题排查

- 端口占用/未开放：
  - 确认本机 5005 已监听：`netstat -ano | findstr 5005`（Windows PowerShell/CMD）
  - 查看容器日志：`docker compose logs -f backend` 是否出现 `Listening for transport dt_socket...`
- 断点灰色空心：
  - 说明类尚未加载，先触发一次对应接口路径；
  - 或源码与运行字节码不一致。由于本项目以 volume 挂载源码并使用 `spring-boot:run + devtools`，保存后会自动编译/重启，通常无需额外路径映射；若仍不匹配，可先执行一次完整构建并重启容器。
- 断点不生效：
  - 检查是否开启了 “Mute Breakpoints”；
  - 检查是否连接到了正确的配置（Host/Port 5005）；
  - Windows 防火墙可能拦截 5005，临时允许或加入入站规则。
- 热更新与热替换：
  - 代码小改动（方法体）可通过 JVM HotSwap 生效；
  - 结构性改动（新增方法/字段等）需要由 Spring DevTools 触发重启，容器已启用。

### 六、附：临时改为“启动即等待调试器”

如需应用启动即挂起等待调试器连接（便于排查早期生命周期问题），将 `docker-compose.yml` 中 JDWP 参数改为：

```text
-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005
```

### 七、附：仅允许本机访问调试端口（可选）

将 compose 的端口映射改成：

```yaml
ports:
  - "127.0.0.1:5005:5005"
```

这样只允许本机连接，增强安全性。


