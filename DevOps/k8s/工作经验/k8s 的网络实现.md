Kubernetes 的网络服务并不直接使用 **CNM**（Container Network Model），而是采用了另一种容器网络模型：**CNI**（Container Network Interface）。以下是它们之间的对比和 Kubernetes 的网络模型说明：

---

### **1. CNM（Container Network Model）**

CNM 是 Docker 的网络插件架构，由 Docker 社区开发和维护，主要由以下三部分组成：

- **网络沙箱（Network Sandbox）**：隔离网络栈，例如 Linux 的网络命名空间。
- **端点（Endpoint）**：连接容器到网络的接口。
- **网络（Network）**：端点的集合，是逻辑上的网络单元。

CNM 是 Docker 原生支持的网络模型，使用 Docker 自带的 **libnetwork**，它支持多种网络模式（桥接、主机模式、自定义网络等）。

---

### **2. CNI（Container Network Interface）**

CNI 是由 CNCF（Cloud Native Computing Foundation）维护的容器网络标准，用于管理容器的网络连接和资源分配。它是 Kubernetes 默认的网络插件模型。CNI 的主要特点：

- 轻量化、专注于容器生命周期的网络操作。
- 标准化的插件接口，支持多种实现方式。

Kubernetes 使用 CNI 插件（如 Flannel、Calico、Weave Net 等）来实现其 Pod 的网络功能，例如 IP 地址分配、网络连接和隔离。

---

### **Kubernetes 与 CNI 的关系**

Kubernetes 默认不使用 CNM，而是选择了 CNI，原因如下：

1. **标准化**：CNI 专为容器编排环境设计，与 Kubernetes 的设计理念更契合。
2. **插件灵活性**：CNI 提供了灵活的接口，可以通过插件实现不同的网络方案（如 Overlay、Underlay 网络）。
3. **生态兼容**：Kubernetes 通过 CNI 支持多种网络插件，例如：
   - **Flannel**：轻量级 Overlay 网络。
   - **Calico**：支持网络策略和 BGP 的路由方案。
   - **Weave Net**：提供简单的 Overlay 网络和加密功能。

---

### **总结**

- Docker 网络使用的是 **CNM**，而 Kubernetes 的网络服务使用的是 **CNI**。
- CNI 为 Kubernetes 提供了一个开放的插件架构，使得 Kubernetes 网络可以高度定制化，支持多种网络方案。

CNI（Container Network Interface）的实现模型也可以理解为围绕几个核心组成部分，但与 CNM 的三要素（沙箱、端点、网络）相比，CNI 更加轻量级和灵活。CNI 的核心关注点是：**如何将网络功能注入到容器中**，并在容器生命周期中管理这些功能。

### **CNI 的核心组成部分**

CNI 模型围绕以下三部分构建：

1. **CNI 插件（CNI Plugins）**  
   - 核心功能：实现具体的网络操作，例如分配 IP 地址、配置路由、设置防火墙规则等。
   - 插件种类：
     - **主插件（Main Plugins）**：负责主要的网络操作，比如 Flannel、Calico、Weave 等。
     - **辅助插件（Chained Plugins）**：处理额外的功能，比如 IP 地址管理、带宽限制、DNS 配置等。
   - 插件通过标准化接口（JSON 格式）与容器运行时通信。
2. **CNI 配置文件（CNI Configuration File）**  
   - 描述了如何加载和使用 CNI 插件。
   - 配置文件路径通常位于 `/etc/cni/net.d/`。
   - 包含以下信息：
     - 网络名称。
     - 插件类型。
     - 网络参数，例如子网范围、网关、路由表等。
   
   **示例配置文件：**
   ```json
   {
     "cniVersion": "0.4.0",
     "name": "mynetwork",
     "type": "bridge",
     "bridge": "cni0",
     "ipam": {
       "type": "host-local",
       "subnet": "10.22.0.0/16",
       "gateway": "10.22.0.1"
     }
   }
   ```
3. **CNI 运行时（Runtime Invocation）**  
   - 由容器运行时（如 Kubernetes 的 kubelet 或 CRI-O）负责调用 CNI 插件。
   - 运行时的职责：
     - **添加网络（ADD）**：在容器创建时为其配置网络。
     - **删除网络（DEL）**：在容器删除时释放网络资源。
   - 通过环境变量传递容器的元数据（如容器的网络命名空间、接口名称）。

---

### **CNI 的工作流程**

CNI 的运行模型分为两个阶段：**网络创建**和**网络删除**。

1. **网络创建（ADD 操作）**
   - 容器运行时调用 CNI 插件，并传递以下信息：
     - 容器的网络命名空间路径（`netns`）。
     - 目标网络配置（来自配置文件）。
     - 容器网络接口名称（`eth0`）。
   - 插件根据配置执行以下步骤：
     1. 在容器网络命名空间中创建虚拟网络接口（veth pair）。
     2. 一端连接到主机的桥接设备或 Overlay 网络，另一端连接到容器。
     3. 分配 IP 地址并配置路由。
2. **网络删除（DEL 操作）**
   - 在容器被销毁时，运行时调用插件的 DEL 方法。
   - 插件释放 IP 地址，删除 veth 接口，清理路由和规则。

---

### **CNI 的实现模型**

可以将 CNI 的实现模型总结为以下几个层次：

#### 1. **底层网络资源**

- 网络命名空间（`Netns`）：隔离容器的网络堆栈。
- 虚拟网络设备（`veth`）：连接容器和主机网络。
- 路由表和 IP 规则。

#### 2. **插件层**

- 每个插件实现一项功能：
  - **主插件**：如 bridge、flannel、calico，完成主要网络功能。
  - **辅助插件**：如 portmap、bandwidth，处理额外需求。

#### 3. **运行时接口**

- 运行时（如 Kubernetes 的 kubelet）通过标准化的 CNI 接口（`ADD`/`DEL`）调用插件完成网络操作。

#### **示意图：**

```plaintext
+------------------------------+
|        CNI 运行时接口        |
|  (通过 ADD/DEL 调用插件)     |
+------------------------------+
           |
           V
+------------------------------+
|         CNI 插件层           |
| - 主插件：bridge, flannel    |
| - 辅插件：portmap, bandwidth |
+------------------------------+
           |
           V
+------------------------------+
| 底层网络资源                 |
| - Netns                     |
| - Veth                      |
| - Routing/IP Rules          |
+------------------------------+
```

---

### **CNI 的优点**

1. **灵活性**：支持多种插件，满足不同场景的需求。
2. **轻量级**：专注于容器网络操作，避免复杂的依赖。
3. **标准化**：使用统一接口，适配多个容器运行时（如 Kubernetes、CRI-O）。

---

### **总结**

CNI 模型围绕 **插件、配置文件和运行时调用接口** 这三个关键部分构建，形成了一个高度模块化、灵活的网络架构。这种设计使得 Kubernetes 能够轻松集成各种网络方案，同时保持高性能和可扩展性。

**CNM（Container Network Model）** 和 **CNI（Container Network Interface）** 是两种独立的容器网络模型和接口，它们并没有直接的上下级关系。实际上，它们是两种不同的容器网络标准，分别由不同的组织开发和维护，旨在解决容器网络的不同方面。

### **CNM 和 CNI 的关系**

- **无直接依赖关系**：CNM 和 CNI 是并行发展的两种标准，没有一个直接依赖于另一个。虽然它们都用于实现容器的网络功能，但它们设计目标、实现方式和适用范围不同。
- **可能共存**：在某些复杂的环境中，CNM 和 CNI 可以协同工作。例如，Docker 使用 CNM 来管理容器网络，但可以通过 CNI 插件来扩展其网络能力。

---

### **区别对比：**

|特性|CNM （Container Network Model）|CNI （Container Network Interface）|
|--|--|--|
|**维护组织**|Docker 社区（libnetwork）|CNCF（Cloud Native Computing Foundation）|
|**适用场景**|Docker 原生网络架构，围绕容器创建和管理网络|专注于容器编排系统（如 Kubernetes）的网络需求|
|**核心组成**|沙箱（Sandbox）、端点（Endpoint）、网络（Network）|插件（Plugin）、配置文件（Config File）、运行时调用接口（Runtime Invocation）|
|**目标**|提供灵活的网络管理抽象，用于 Docker 容器的网络通信|为容器生命周期管理提供标准化网络接口，支持多种网络插件|
|**网络插件支持**|支持 libnetwork 插件|支持多种 CNI 插件（如 Flannel、Calico、Weave Net 等）|
|**典型应用**|Docker 默认的网络模型，包含 bridge、host、overlay 等模式|Kubernetes 网络模型，负责 Pod 的网络配置|
|**接口灵活性**|偏向 Docker 原生，扩展性较低|提供标准接口，插件生态丰富|

---

### **为什么说 CNI 不是 CNM 的底层**

1. **设计理念不同**  
   - CNM 更偏向于网络架构的抽象，定义了网络的基本构件（沙箱、端点和网络）以及这些构件之间的关系。
   - CNI 更专注于容器生命周期中的具体网络操作（如 IP 分配、路由配置等），为实现这些操作提供一个轻量化接口。
2. **实现独立性**  
   - CNM 使用的是 **libnetwork**（Docker 内置的网络库）实现网络功能，而 CNI 是一个独立的标准，与 Docker 无直接依赖。
   - CNI 的插件可以直接操作 Linux 网络堆栈，完全不需要 CNM 的帮助。
3. **不同的生态**  
   - CNM 专为 Docker 设计，其生态围绕 Docker 网络需求构建。
   - CNI 是为 Kubernetes 设计的，提供的插件生态（如 Flannel、Calico）完全独立于 CNM。

---

### **CNM 和 CNI 如何共存**

尽管 CNM 和 CNI 独立发展，但在一些场景下它们可以互补。例如：

- Docker 可以通过 CNM 管理网络，但借助 **CNI 插件** 提供更高级的网络功能，例如复杂的 Overlay 网络、网络策略或 BGP 路由。
- Kubernetes 使用 CNI 作为默认网络模型，但如果运行在 Docker 环境下，Docker 内部仍然使用 CNM 来管理容器的基本网络接口。

---

### **总结**

1. CNM 和 CNI 是两个并行的容器网络标准，**CNI 并不是 CNM 的底层**，两者独立实现。
2. CNM 适用于 Docker 的网络管理，CNI 适用于 Kubernetes 和其他编排系统。
3. 在一些复杂场景中，它们可以通过插件的方式协同工作。
