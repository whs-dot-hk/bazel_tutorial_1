load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

http_archive(
    name = "io_bazel_rules_docker",
    sha256 = "4521794f0fba2e20f3bf15846ab5e01d5332e587e9ce81629c7f96c793bb7036",
    strip_prefix = "rules_docker-0.14.4",
    urls = ["https://github.com/bazelbuild/rules_docker/releases/download/v0.14.4/rules_docker-v0.14.4.tar.gz"],
)

load(
    "@io_bazel_rules_docker//repositories:repositories.bzl",
    container_repositories = "repositories",
)

container_repositories()

load("@io_bazel_rules_docker//repositories:deps.bzl", container_deps = "deps")

container_deps()

load("@io_bazel_rules_docker//repositories:pip_repositories.bzl", "pip_deps")

pip_deps()

load(
    "@io_bazel_rules_docker//container:container.bzl",
    "container_pull",
)

container_pull(
    name = "debian_image",
    digest = "sha256:e7157902df9c7549eea5eb7b896cdca02d917e2ba0e339cd4a5087e2b53eb1d7",  # 9.13
    registry = "docker.io",
    repository = "debian",
)

load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_file")

http_file(
    name = "kubectl_binary",
    downloaded_file_path = "kubectl",
    sha256 = "da4de99d4e713ba0c0a5ef6efe1806fb09c41937968ad9da5c5f74b79b3b38f5",
    urls = [
        "https://storage.googleapis.com/kubernetes-release/release/v1.19.1/bin/linux/amd64/kubectl",
    ],
)
