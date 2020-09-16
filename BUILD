load("@io_bazel_rules_docker//container:container.bzl", "container_image", "container_layer")
load("@io_bazel_rules_docker//docker/util:run.bzl", "container_run_and_commit")

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

container_image(
    name = "my_kubectl_image",
    base = ":kubectl_install_commit.tar",
)

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
