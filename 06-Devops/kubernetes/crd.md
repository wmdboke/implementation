### 一、 运行demo

###### 安装

```
git clone https://github.com/kubernetes-sigs/kubebuilder
cd kubebuilder
make build
cp bin/kubebuilder $GOPATH/bin
```

###### 配置环境变量

```bash
export GO111MODULE=on
export GOPROXY=https://goproxy.io

//网上搜的， 下面的安装我是安装失败的。我直接 make 采用go自动下载的依赖 
go install sigs.k8s.io/kustomize/v3/cmd/kustomize
```

###### 创建project

```bash
kdir $GOPATH/src/demo
cd $GOPATH/src/demo
kubebuilder init --domain demo.com --license apache2 --owner "cwb"
```

文件夹目录：

```bash
tree .
.
├── Dockerfile
├── Makefile
├── PROJECT
├── bin
│   └── manager
├── config
│   ├── certmanager
│   │   ├── certificate.yaml
│   │   ├── kustomization.yaml
│   │   └── kustomizeconfig.yaml
│   ├── default
│   │   ├── kustomization.yaml
│   │   ├── manager_auth_proxy_patch.yaml
│   │   ├── manager_webhook_patch.yaml
│   │   └── webhookcainjection_patch.yaml
│   ├── manager
│   │   ├── kustomization.yaml
│   │   └── manager.yaml
│   ├── prometheus
│   │   ├── kustomization.yaml
│   │   └── monitor.yaml
│   ├── rbac
│   │   ├── auth_proxy_role.yaml
│   │   ├── auth_proxy_role_binding.yaml
│   │   ├── auth_proxy_service.yaml
│   │   ├── kustomization.yaml
│   │   ├── leader_election_role.yaml
│   │   ├── leader_election_role_binding.yaml
│   │   └── role_binding.yaml
│   └── webhook
│       ├── kustomization.yaml
│       ├── kustomizeconfig.yaml
│       └── service.yaml
├── go.mod
├── go.sum
├── hack
│   └── boilerplate.go.txt
└── main.go
```

###### 创建CRD

```
// 创建 crd 的配置文件
kubebuilder create api --group infra --version v1 --kind VirtulMachine
// 直接将 crd 在k8s内安装
make install # 安装CRD
# kubectl get crd
NAME                                           AGE
virtulmachines.infra.demo.com                  52m
```

###### 运行程序 controller

```bash
go run main.go
//或者 make run # 启动controller

2019-12-28T00:22:09.789+0800    INFO    controller-runtime.metrics  metrics server is starting to listen    {"addr": ":8080"}
2019-12-28T00:22:09.789+0800    INFO    setup   starting manager
2019-12-28T00:22:09.790+0800    INFO    controller-runtime.manager  starting metrics server {"path": "/metrics"}
```

###### 创建一个crd实例

```
kubectl apply -f config/samples/xxx.yaml
```

### 二、 开发需求

###### 配置参数

- 结构体参数定义`api/v1/virtulmachine_types.go`
  
  ```
  type VirtulMachineSpec struct {
      // INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
      // Important: Run "make" to regenerate code after modifying this file
  
      // Foo is an example field of VirtulMachine. Edit VirtulMachine_types.go to remove/update
      Foo string `json:"foo,omitempty"`
      Name string `json:"name,omitempty"`
      // 定义所需参数
  }
  ```

- 配置文件yaml设置参数 `config/samples`
  
  ```
  apiVersion: infra.qipajun.com/v1
  kind: VirtulMachine
  metadata:
  name: virtulmachine-sample
  spec:
  # Add fields here
  foo: bar
  name: "cc"
  ```

- 实现相关事件处理 : `Reconcile`接口负责实现controller事件监听处理
  
  ```
  func (r *VirtulMachineReconciler) Reconcile(req ctrl.Request) (ctrl.Result, error) {
      ctx := context.Background()
      log := r.Log.WithValues("virtulmachine", req.NamespacedName)
  
      // your logic here
      vm := &infrav1.VirtulMachine{}
      if err := r.Get(ctx, req.NamespacedName, vm); err != nil {
          log.Error(err, "unable to get vm")
      } else {
          fmt.Println("Get vm spec info success, ", vm.Spec.Name)
      }
  
      // 2. 更新虚拟机状态
      vm.Status.UpdateLastTime = metav1.Now()
      vm.Status.Status = "Running"
      if err := r.Status().Update(ctx, vm); err != nil {
          log.Error(err, "not update vm  status.")
      }
  
      // 3. 删除虚拟机
      //time.Sleep(time.Second * 5)
      //if err := r.Delete(ctx, &vm); err != nil {
      //    log.Error(err, "unable to delete vm ", "vm", vm)
      //}
  
      return ctrl.Result{}, nil
  }
  ```

- 更新、构建运行

```
make && make install && make run
kubectl apply -f config/samples/
```

###### 更新状态

```go
type VirtulMachineStatus struct {
    // INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
    // Important: Run "make" to regenerate code after modifying this file

    UpdateLastTime metav1.Time `json:"update_last_time"`
    Status         string      `json:"status"`
}
```

<font color=#ff0000>修改后需要重新 make&make install && make run</font>
