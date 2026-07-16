# stuck_recovery_controller

`/control/command/nominal_control_cmd` と`/vehicle/status/velocity_status` をsubscribeし、スタックを検知したら直進で後退する機能。
スタックを検知していない時は、`/control/command/nominal_control_cmd`を`/control/command/control_cmd`にそのままpublishする。

## Usage

[aichallenge-racingkart](https://github.com/AutomotiveAIChallenge/aichallenge-racingkart) から利用する場合は以下の手順で組み込む。

### 1. vcs importでこのリポジトリを取り込む

`autoware.repos` に本リポジトリを追加し、`aichallenge/workspace/src/aichallenge_submit/stuck_recovery_controller` にvcs importする。

```yaml
repositories:
  aichallenge/workspace/src/aichallenge_submit/stuck_recovery_controller:
    type: git
    url: https://github.com/AutomotiveAIChallenge/stuck_recovery_controller.git
    version: main
```

`aichallenge_submit_launch/package.xml` に依存を追加する。

```xml
<exec_depend>stuck_recovery_controller</exec_depend>
```

### 2. コントローラの出力をこのノード経由にremapする

`pure_pursuit.launch.xml` / `mpc.launch.xml` の出力先を、直接 `/control/command/control_cmd` にpublishするのではなく `/control/command/nominal_control_cmd` にremapし、本ノードが最終的な `/control/command/control_cmd` をpublishするようにする。

各launchファイルに出力先を切り替えられる引数を追加する。

```xml
<!-- pure_pursuit.launch.xml -->
<arg name="output_control_cmd" default="/control/command/control_cmd"/>
...
<remap from="output/control_cmd" to="$(var output_control_cmd)"/>
```

```xml
<!-- mpc.launch.xml -->
<arg name="output_control_cmd" default="/control/command/control_cmd"/>
...
<remap from="/control/command/control_cmd" to="$(var output_control_cmd)"/>
```

`reference.launch.xml` から `output_control_cmd` に `/control/command/nominal_control_cmd` を渡し、本ノードを起動する。

```xml
<include file="$(find-pkg-share aichallenge_submit_launch)/launch/control/pure_pursuit.launch.xml">
  ...
  <arg name="output_control_cmd" value="/control/command/nominal_control_cmd"/>
</include>

<include file="$(find-pkg-share aichallenge_submit_launch)/launch/control/mpc.launch.xml">
  ...
  <arg name="output_control_cmd" value="/control/command/nominal_control_cmd"/>
</include>

<node pkg="stuck_recovery_controller" exec="stuck_recovery_controller_node" name="stuck_recovery_controller" output="screen">
  <param name="use_sim_time" value="$(var use_sim_time)"/>
</node>
```
