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
In bazel, we specified the `kubectl` version in `WORKSPACE`
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

```sh
$ bazel run //:my_kubectl_image
$ docker run -it bazel:my_kubectl_image sh -c "kubectl version --client" 
```

## Building without docker
Stop dockerd and try
```sh
$ bazel build //:my_better_kubectl_image
```

We make `kubectl` executable outside docker and "copy" it into the `files` attribute of `container_image` (could also be a new `container_layer` with the `layers` attribute, like above)
```starlark
genrule(
    name = "kubectl_executable",
    srcs = ["@kubectl_binary//file"],
    outs = ["my_kubectl"],
    cmd = "cp $(location @kubectl_binary//file) $@ && chmod a+x $@",
)

container_image(
    name = "my_better_kubectl_image",
    base = "@debian_image//image",
    files = [
        ":kubectl_executable",
    ],
    symlinks = {
        "/usr/local/bin/kubectl": "/my_kubectl",
    },
)
```

Start dockerd again, to run the new image
```sh
$ bazel run //:my_better_kubectl_image
$ docker run -it bazel:my_better_kubectl_image sh -c "kubectl version --client"
```
