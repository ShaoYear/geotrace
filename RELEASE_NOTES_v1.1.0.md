# GeoTrace v1.1.0

发布日期：2026-09-13

GeoTrace v1.1.0 是一次面向正式发布的可靠性、实时反馈和交互流畅性更新。本版本保持现有 GPS 记录、轨迹匹配、照片 EXIF、GPX 和本地处理流程不变，重点补齐数据一致性并改善地图页面体验。

## 重点更新

### 实时轨迹地图

- 轨迹记录页面新增实时高德地图，位于时间、距离、定位点和速度指标上方。
- 开始记录后实时显示当前位置，轨迹线随着 GPS 点持续延伸。
- 默认自动跟随当前位置；用户主动拖动或缩放后交还地图控制权。
- 支持「回到当前位置」恢复跟随，并支持点击进入大地图模式。
- 暂停、定位暂时丢失和停止时保留已有轨迹。
- 与轨迹详情复用同一套高德 MapView、坐标转换和 Polyline/Marker 方案。

### 数据一致性与照片安全

- 改善 Photo Picker、SAF 和 MediaStore Uri 的读取与重试。
- COPY 使用 MediaStore `IS_PENDING`，失败、空文件或发布失败时自动清理临时副本。
- 多次复制照片时使用唯一输出名，避免同名覆盖和历史结果互相去重。
- EXIF 写入失败时清理新副本；数据库提交失败时执行文件补偿清理。
- 明确区分原始 `sourceUri`、输出 `fileUri`、`realPath` 和文件名，照片身份不再依赖文件名。

### 轨迹和数据库稳定性

- 过滤乱序 GPS 点，串行写入实时轨迹点。
- 修复轨迹匹配的亚秒级边界问题。
- 使用数据库聚合查询照片摘要，减少大数据量场景下的重复读取。
- Room schema 升级到 v6，并增加照片查询索引。

### 地图返回流畅性

- 缩短 Navigation3 进入/返回过渡，进入 180ms、返回 160ms。
- 调整 MapView 生命周期释放顺序，减少地图页面返回时的瞬时卡顿。
- 导航返回不再同步保存一次临时地图状态。
- 移除照片详情页重型的全量地图清理，改为按页面清理实际创建的覆盖物。
- Bitmap 回收移至后台调度；手动定位页离开时取消搜索和逆地理任务。

### 克制型 Glass UI 基础

- 轻半透明表面、细边界、低阴影和非常淡的背景环境光晕。
- 统一页面留白、Section 间距、圆角、按钮反馈和文字层级。
- 列表和照片网格保留滚动位置，改善页面返回后的连续感。
- 保留地图和照片的内容重点，减少过度透明、发光和厚重卡片。

## 技术说明

- 实时轨迹绘制使用本地 GPS 点和高德 Android SDK 的 Polyline/Marker。
- 不会因为每个 GPS 点调用路径规划、逆地理编码或其他高德 Web API。
- 地图显示端可进行轻量抽稀和增量更新，但 Room 中的原始轨迹点不会被删除或替换。
- 高德地图瓦片仍会按地图显示需要联网加载，具体配额和服务政策以高德控制台及官方说明为准。
- GPS Foreground Service、Room TrackPoint、GPX、EXIF 匹配和轨迹详情业务逻辑保持不变。

## 版本信息

| 项目 | 值 |
|---|---|
| applicationId | `com.shaoyear.geotrace` |
| versionName | `1.1.0` |
| versionCode | `2` |
| minSdk | `26` |
| targetSdk / compileSdk | `36 / 36` |
| 数据库 | Room v6 |

## 验证与发布前检查

- 已完成变更文件的定向 Kotlin 编译检查。
- 已完成源码和测试源码结构检查。
- Gradle 全量构建在当前 Windows/JDK 环境建立 loopback 连接时失败，未进入源码编译。上传 APK 前请在 Android Studio 或可用 Gradle 环境执行：

```powershell
.\gradlew.bat testDebugUnitTest --console=plain
.\gradlew.bat assembleRelease --console=plain
```

- 发布 APK 前请在真机验证：实时记录 10 分钟以上、暂停/继续、地图拖动后不强制回拉、全屏地图、轨迹详情一致性、照片 COPY/EXIF、地图页面返回流畅性。

## 已知边界

- 覆盖原照片模式目前是系统授权后直接写入可写 Uri，尚未完成 cache staged 原子替换的专项真机验证。
- 高德 Key 需要在高德开放平台登记 `com.shaoyear.geotrace` 及对应签名 SHA1。

## 安装

下载本 Release 附件中的 `GeoTrace-v1.1.0.apk`，Android 8.0（API 26）及以上设备可安装。

首次使用地图或地点搜索前，请确保已配置有效的高德 Key，并按系统提示授予定位、通知和照片访问权限。
