## Ansible
The ansible below install `kubectl` version `v1.19.1` into `/usr/local/bin/kubectl` with file mode `0755`.
```yaml
- become: "yes"
  get_url:
    dest: /usr/local/bin/kubectl
    mode: "0755"
    url: >-
      https://storage.googleapis.com/kubernetes-release/release/v1.19.1/bin/linux/amd64/kubectl
  name: Install kubectl
```

## Bazel
We specified the `kubectl` version in `WORKSPACE`
```starlark
WORKSPACE
---
...

http_file(
    name = "kubectl_binary",
    downloaded_file_path = "kubectl",
    sha256 = "da4de99d4e713ba0c0a5ef6efe1806fb09c41937968ad9da5c5f74b79b3b38f5",
    urls = [
        "https://storage.googleapis.com/kubernetes-release/release/v1.19.1/bin/linux/amd64/kubectl",
    ],
)
```

And put it under `/usr/local/bin/kubectl` with the right file mode
```starlark
BUILD
---
...

container_layer(
    name = "kubectl_layer",
    files = [
        "@kubectl_binary//file",
    ],
)

container_image(
    name = "kubectl_installer",
    base = "@debian_image//image",
    cmd = "",
    entrypoint = "",
    layers = [":kubectl_layer"],
)

container_run_and_commit(
    name = "kubectl_install",
    commands = [
        "chmod a+x /kubectl",
        "cp /kubectl /usr/local/bin/kubectl",
        "rm /kubectl",
    ],
    image = ":kubectl_installer.tar",
)
```

```starlark
$ bazel run //:my_kubectl_image
$ docker run -it bazel:my_kubectl_image sh -c "kubectl version --client" 
```
