#简介

易于配置的、高并发带宽的、可扩展的、ServerToServer(P2P)的、加密的，TCP隧道工具。

#设计

本设计类似 Open Shortest Path First，OSPF。 但或许更适合称为 Open Fastest Path First，OTPF —— 开放式最快路径优先

- 节点可以部署任意多个，并随时追加
- 所有节点的配置文件位于一个网址而不是本地文件。好处是只需要修改一处文件，即可修改所有节点的配置，而不需要维护所有的节点服务器
- 配置文件可以被加密，由节点启动时配置的key进行解密，保证配置文件的安全性
- 任何节点均同时承担监听角色（Listener）和连接角色（Connector）
- 节点会被分组，例如墙内（inGFW）和墙外(outGFW)
- 节点启动时会检查配置文件和配置文件版本是否有更新
- 节点启动时会检查是否可执行程序有更新
- 需要更新的可执行程序也是被加密的
- 节点启动时会向其他节点确认自己的角色和职责
- 当一个节点收到请求时，会同时向所有其他的节点发出请求（广播）
- 如果是跨组别的节点，其间数据会被加密
- 如果是跨组别的节点，其间数据包会被同时在节点间已经建立的多个通道上发送多次
- 最快与目标地址建立连接的连接角色（Connector）节点，会被确认为本次连接的指定连接角色（Connector）节点
- 一旦指定连接角色（Connector）节点被确认后，其后尽管一个节点收到请求时，会同时向所有其他的节点发出请求（广播），但最终所有节点都会发送请求到指定连接角色（Connector）节点
- 指定连接角色（Connector）节点有责任不重复发送相同的数据包
- 监听角色（Listener）节点总是将最快返回的数据，优先返回给用户

设计优点是可以让两点见不论网络情况多负责，其间可以达到最快的速度。缺点是会消耗额外的带宽。

##数据包结构设计

数据包通讯协议，使用FlatBuffers。

- 节点版本号
- 连接ID，由监听角色（Listener）节点根据本机信息和时间生成
- 数据包ID，递增
- 数据包目的地节点：监听角色（Listener）节点或指定的连接角色（Connector）节点，或节点组别。
- 数据包发起者节点ID
- 已经经过的节点列表（包括本节点）
- 数据包属性：是否加密，是上行包还是下行包，是否压缩
- 数据包数据内容
