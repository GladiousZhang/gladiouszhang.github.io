---
title: k8s实现pod内部的webshell
date: 2024-08-13 15:58:27
tags:
  - 分布式系统
  - 实习项目
---

<!-- toc -->

# 背景介绍

这是实习的时候，公司让我开发的一个功能，主要希望实现的是前端与k8s中的pod的webshell功能，并且包含tab代码补全的功能。基于https://github.com/GanonYou/k8s-webshell-gin的代码来实现的。但是实现流程不一样：源代码是自己暴露出ws接口，等待前端连接），我的代码是集群先和后端建立ws，然后等待前端连接后端，然后前端先把消息传到后端，后端经过鉴权再转发给集群。

在本文档里，大致解释一下集群上的webshell（相当于是后端）的相关代码。

# 代码介绍

## `handleConnections`

```go
sshReq = ClientSet.CoreV1().RESTClient().Post().
		Resource("pods").
		Name(pod).
		Namespace(namespace).
		SubResource("exec").
		VersionedParams(&v1.PodExecOptions{
			Container: container,
			Command:   []string{"bash"},
			Stdin:     true,
			Stdout:    true,
			Stderr:    true,
			TTY:       true,
		}, scheme.ParameterCodec)
```

以上只是用于创建一个请求，还未发送。

`ClientSet`是一个指向 `kubernetes.Clientset` 类型的实例的指针，CoreV1提供了对 Kubernetes Core API 组，例如 Pods、Nodes、Services 等资源的访问，`RESTClient()`返回一个 `rest.Interface` 对象，允许直接使用 REST API 请求而不通过高层次的抽象。这个对象提供了更底层的 API 操作。最后，`Post()`用于构造一个 HTTP POST 请求。这个请求会用来与 Kubernetes API 交互，通常用于创建或更新资源。

`SubResource("exec")` 表示我们要操作的是 Pod 的 exec 子资源，这通常用于执行命令。即，我们希望在指定的 Pod 内执行某个命令。

`VersionedParams()`用于设置请求的参数，并且按照 API 版本对这些参数进行编码。

- `v1.PodExecOptions`中：
  - **`Container`**: 要执行命令的容器名。
  - **`Command`**: 要在容器中执行的命令数组。这里是 `[]string{"bash"}`，表示在容器中启动一个 Bash shell。
  - **`Stdin`, `Stdout`, `Stderr`**: 这些布尔值表示是否需要连接标准输入、标准输出和标准错误。
  - **`TTY`**: 布尔值，表示是否需要终端

``` go
// 创建到容器的连接
	if executor, err = remotecommand.NewSPDYExecutor(restConf, "POST", sshReq.URL()); err != nil {
		goto END
	}
```

初始化一个 `remotecommand.Executor` 对象，以便能够通过 Kubernetes API 执行命令。创建 `executor` 的过程涉及配置 API 连接、指定 HTTP 方法（`POST`）以及构造执行请求的 URL。

**`sshReq.URL()`**:

- `sshReq` 是之前构建的 `*rest.Request` 对象，包含了 Kubernetes API 的请求信息。
- `sshReq.URL()` 返回一个 `*url.URL` 对象，表示请求的完整 URL。这个 URL 指向 Kubernetes API 的 exec 子资源，用于执行命令。
- 例如，这个 URL 可能类似于 `https://kubernetes/api/v1/namespaces/default/pods/my-pod/exec?stdin=true&stdout=true&stderr=true`。

```go
	handler = &streamHandler{wsConn: wsConn}
	ctx = context.Background()

	if err = executor.StreamWithContext(ctx, remotecommand.StreamOptions{
		Stdin:  handler,
		Stdout: handler,
		Stderr: handler,
		// TerminalSizeQueue: handler,
		Tty: true,
	}); err != nil {
		goto END
	}
```

将websocket直接连接到容器内的Stdin等。其实我修改后可以不用streamHandler，直接给我们的websocket加上io.read和write的接口就行，但是现在还没改。
