# Code Style

The code should adhere to the [Dart code style](https://dart.dev/effective-dart/style) with a 
maximum line length of 80 characters. 
Moreover, code should fulfill all LINT rules set in `analysis_options.yaml`. Most IDEs, 
such as Android Studios, automatically provide warnings if certain rules are violated.

Auto-format your code using the following command inside the project:
```sh
  dart format .
```

To get an automatic overview of all code style related issues, use:
```sh
  flutter analyze
```

Some issues can be fixed automatically. To do so, use:
```sh
  dart fix --apply
```

> In the future, we will automatically check code style with 
> [GitHub Actions](Continuous-Integration-Pipelines.md), rejecting pull requests which do not follow
> the defined rules.
{style="warning"}

Moreover, this code must follow architectural concepts described in 
[base architecture reference](Base-Architecture.md).
All logical elements must provide unit tests.