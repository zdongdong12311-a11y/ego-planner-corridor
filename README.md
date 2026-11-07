# EGO-Planner 实机高度走廊说明

本版本在原版 EGO-Planner 的基础上增加了“虚拟高度走廊”，并提供了 Fast-LIO 实机启动文件。

## 新增功能

- 虚拟地板和天花板：规划轨迹会避开设定高度范围以外的区域。
- 目标高度限制：2D Nav Goal 的目标点会自动使用设定飞行高度，并限制在走廊内。
- 点云高度过滤：忽略走廊外的 Fast-LIO 点云，减少地图中的地面、天花板点。
- 实机启动：`run_in_exp.launch` 直接接入 Fast-LIO 的 `/Odometry` 和 `/cloud_registered`，不会启动仿真器。

## 高度参数

默认配置：

| 参数 | 默认值 | 含义 |
| --- | ---: | --- |
| `ground_height` | `0.0` | Fast-LIO 坐标系中的地面 Z 值 |
| `min_flight_height` | `0.30` | 距地最小飞行高度 |
| `max_flight_height` | `2.20` | 距地最大飞行高度 |
| `fsm/default_target_height` | `1.0` | 2D Nav Goal 使用的目标高度 |

因此默认走廊为 `0.30 m ≤ z < 2.20 m`，2D Nav Goal 的目标高度为 `1.0 m`。

`ground_height` 必须按 Fast-LIO 坐标填写。若 Fast-LIO 启动时雷达离地约 `0.4 m`，地面通常约为 `-0.4`，启动时应设置 `ground_height:=-0.4`。

## 实机启动

先启动 Fast-LIO，确认它持续发布：

```bash
/Odometry
/cloud_registered
```

再启动规划器：

```bash
roslaunch ego_planner run_in_exp.launch
```

例如地面在 Fast-LIO 坐标中为 `-0.4 m`：

```bash
roslaunch ego_planner run_in_exp.launch ground_height:=-0.4
```

默认点云 frame 是 `camera_init`。如果 `/cloud_registered` 的 `header.frame_id` 不同，可覆盖：

```bash
roslaunch ego_planner run_in_exp.launch map_frame:=你的点云frame
```

## 2D Nav Goal 使用方法

1. RViz 的 Fixed Frame 设置为 `map_frame`，默认是 `camera_init`。
2. 点击工具栏的 **2D Nav Goal**，在地图中选择目标位置和朝向。
3. 终端出现 `Triggered!` 后，规划器会以默认高度 `1.0 m` 生成轨迹。
4. 轨迹会发布到 `/planning/bspline`，轨迹服务器会生成 `/position_cmd`。

## RViz 中的颜色

- 蓝色絮状体素：Fast-LIO 点云经 0.1 m 栅格化和障碍物膨胀后的局部障碍物地图。
- 粉色平面：虚拟天花板，表示最大高度以上的不可飞区域。

如需避免仿真地图干扰，请在 RViz 中关闭 `simulation_map`，查看 `map inflate`。

## 注意

- 高度走廊限制的是规划器参考点；实际机体仍需预留机身尺寸和定位误差。
- 初始起飞阶段会暂时放宽最低高度限制，便于从地面进入走廊。
- `/position_cmd` 是控制指令话题。接入飞控前，应先在 RViz 中确认目标点、障碍物和轨迹都正确。
