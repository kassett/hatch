# How to select the installer

-----

## Enabling UV

The [virtual](../../plugins/environment/virtual.md) environment type by default uses [virtualenv](https://github.com/pypa/virtualenv) for virtual environment creation and [pip](https://github.com/pypa/pip) to install dependencies. You can speed up environment creation and dependency resolution by using [UV](https://github.com/astral-sh/uv) instead of both of those tools.

!!! warning "caveat"
    UV is under active development and may not work for all dependencies.

To do so, enable the `uv` [option](../../plugins/environment/virtual.md#options). For example, if you wanted to enable this functionality for your [test environments](../../config/internal/testing.md#customize-environment), you could set the following:

```toml config-example
[tool.hatch.envs.hatch-test]
uv = true
```

The next time you use environments, Hatch will create a dedicated environment for UV (if it does not yet exist) which will be used for all subsequent environment creation and dependency resolution & installation.

!!! tip
    All environments that enable UV will have `uv` available on PATH.

## Configuring the version

The UV that is shared by all environments uses a specific version that is known to work with Hatch. If you want to use a different version, you can override the [dependencies](../../config/environment/overview.md#dependencies) for the internal `hatch-uv` environment:

```toml config-example
[tool.hatch.envs.hatch-uv]
dependencies = [
  "uv>9000",
]
```

## Externally managed

If you want to manage UV yourself, you can expose it to Hatch by setting the `HATCH_ENV_TYPE_VIRTUAL_UV_PATH` environment variable. This should be the absolute path to a UV binary which Hatch will use instead of the internal environment. This implicitly [enables](#enabling-uv) the `uv` option.

## Installer script alias

If you have [scripts](../../config/environment/overview.md#scripts) or [commands](../../config/environment/overview.md#commands) that call `pip`, it may be useful to alias the `uv pip` command to `pip` so that you can use the same commands for both methods of configuration and retain your muscle memory. The following is an example of a matrix that [conditionally](../../config/environment/advanced.md#option-overrides) enables UV and sets the alias:

```toml config-example
[[tool.hatch.envs.example.matrix]]
installer = ["uv", "pip"]

[tool.hatch.envs.example.overrides]
matrix.installer.uv = [
  { value = true, if = ["uv"] },
  { value = false, if = ["pip"] },
]
matrix.installer.scripts = [
  { key = "pip", value = "uv pip {args}", if = ["uv"] },
]
```

Another common use case is to enable UV for all [test environments](../../config/internal/testing.md). In this case, you often wouldn't want to modify the `scripts` mapping directly but rather add an [extra script](../../config/environment/overview.md#extra-scripts):

```toml config-example
[tool.hatch.envs.hatch-test]
uv = true

[tool.hatch.envs.hatch-test.extra-scripts]
pip = "uv pip {args}"
```
