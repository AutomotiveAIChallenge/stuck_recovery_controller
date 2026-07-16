# stuck_recovery_controller

`/control/command/nominal_control_cmd` と`/vehicle/status/velocity_status` をsubscribeし、スタックを検知したら直進で後退する機能。
スタックを検知していない時は、`/control/command/nominal_control_cmd`を`/control/command/control_cmd`にそのままpublishする。

## Usage

[aichallenge-racingkart](https://github.com/AutomotiveAIChallenge/aichallenge-racingkart) から利用する場合は以下の手順で組み込む。

### 1. git cloneでこのリポジトリを取り込む

`aichallenge/workspace/src/aichallenge_submit/stuck_recovery_controller` にcloneする。

```bash
git clone https://github.com/AutomotiveAIChallenge/stuck_recovery_controller.git \
  aichallenge/workspace/src/aichallenge_submit/stuck_recovery_controller
```

### 2. コントローラの出力をこのノード経由にremapする

ここでは例としてpure pursuitで説明する。

出力先を直接 `/control/command/control_cmd` にpublishするのではなく `/control/command/nominal_control_cmd` にremapし、本ノードが最終的な `/control/command/control_cmd` をpublishするようにする。

`pure_pursuit.launch.xml` に出力先を切り替えられる引数を追加する。

```xml
<!-- pure_pursuit.launch.xml -->
<arg name="output_control_cmd" default="/control/command/control_cmd"/>
...
<remap from="output/control_cmd" to="$(var output_control_cmd)"/>
```

`reference.launch.xml` から `output_control_cmd` に `/control/command/nominal_control_cmd` を渡し、本ノードを起動する。

```xml
<include file="$(find-pkg-share aichallenge_submit_launch)/launch/control/pure_pursuit.launch.xml">
  ...
  <arg name="output_control_cmd" value="/control/command/nominal_control_cmd"/>
</include>

<node pkg="stuck_recovery_controller" exec="stuck_recovery_controller_node" name="stuck_recovery_controller" output="screen">
  <param name="use_sim_time" value="$(var use_sim_time)"/>
</node>
```
