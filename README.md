# icon_font_generator

[![pub package](https://img.shields.io/pub/v/icon_font_generator.svg)](https://pub.dartlang.org/packages/icon_font_generator)

The icon_font_generator package provides an easy way to convert SVG icons to OpenType font
and generate Flutter-compatible class that contains identifiers for the icons 
(just like [CupertinoIcons][] or [Icons][] classes).

The package is written fully in Dart and doesn't require any external dependency.
Compatible with dart2js and dart2native.

[CupertinoIcons]: https://api.flutter.dev/flutter/cupertino/CupertinoIcons-class.html
[Icons]: https://api.flutter.dev/flutter/material/Icons-class.html

## Font generation

### Install via dev dependency 通过 dev 项目依赖安装

```shell
$ flutter pub add --dev icon_font_generator

# And it's ready to go:
$ flutter pub run icon_font_generator:generator <input-svg-dir> <output-font-file> [options]
```

### or [Globally activate][] the package:

[globally activate]: https://dart.dev/tools/pub/cmd/pub-global

```shell
$ pub global activate icon_font_generator

# And it's ready to go:
$ icon_font_generator <input-svg-dir> <output-font-file> [options]
```

### Friendly reminder 友情提示
>
> 如果你的svg文件是 evenodd, 且 icon 显示不正确, 请安装 [picosvg https://github.com/googlefonts/picosvg](https://github.com/googlefonts/picosvg) 并使用下面的脚本
> If your svg file is evenodd, and the icon is not displayed correctly, please install [picosvg https://github.com/googlefonts/picosvg](https://github.com/googlefonts/picosvg) and use the following script

```shell

```sh

  sudo chmod +x ./picosvg.sh  && ./picosvg.sh
  <!-- or -->
  sh ./picosvg.sh

```

> 👇 ./picosvg.sh

```sh
#!/bin/bash
# 检查是否安装了 picosvg
if ! command -v picosvg &> /dev/null
then
    echo "picosvg 未安装，正在安装..."
    pip3 install picosvg
    if [ $? -ne 0 ]; then
        echo "picosvg 安装失败，请手动安装 picosvg。"
        exit 1
    fi
fi

# 输入和输出目录
input_dir="./assets/fonts/svg"
output_dir="./assets/fonts/fix"

# 确保输出目录存在
mkdir -p "$output_dir"

# 遍历输入目录中的所有 SVG 文件
for svg_file in "$input_dir"/*.svg; do
    # 提取文件名，不带路径
    file_name=$(basename "$svg_file")
    
    # 使用 picosvg 处理并输出到目标目录
    picosvg "$svg_file" > "$output_dir/$file_name"
    
    # 打印处理的信息（可选）
    echo "Processed $file_name"
done
```

Required positional arguments:
- `<input-svg-dir>`
Path to the input directory that contains .svg files.
- `<output-font-file>`
Path to the output font file. Should have .otf extension.

Flutter class options:
- `-o` or `--output-class-file=<path>`
Output path for Flutter-compatible class that contains identifiers for the icons.
- `-c` or `--class-name=<name>`
Name for a generated class.
- `-p` or `--package=<name>`
Name of a package that provides a font. Used to provide a font through package dependency.
- `--[no-]format`
Format dart generated code.

Font options:
- `-f` or `--font-name=<name>`
Name for a generated font.
- `--[no-]normalize`
Enables glyph normalization for the font.
Disable this if every icon has the same size and positioning.
(defaults to on)
- `--[no-]ignore-shapes`
Disables SVG shape-to-path conversion (circle, rect, etc.).
(defaults to on)

Other options:
- `-z` or `--config-file=<path>`
Path to icon_font_generator yaml configuration file.
pubspec.yaml and icon_font.yaml files are used by default.
- `-r` or `--recursive`
Recursively look for .svg files.
- `-v` or `--verbose`
Display every logging message.
- `-h` or `--help`
Shows usage information.

*Usage example:*

```shell
$ icon_font_generator assets/svg/ fonts/my_icons_font.otf --output-class-file=lib/my_icons.dart -r
```

Updated Flutter project's pubspec.yaml:

```yaml
flutter:
  fonts:
    - family: My Icons
      fonts:
        - asset: fonts/my_icons_font.otf
```

## Config file

icon_font_generator's configuration can also be placed in yaml file.
Add _icon_font_ section to either `pubspec.yaml` or `icon_font.yaml` file:

```yaml
icon_font:
  input_svg_dir: "assets/fonts/fix/"
  output_font_file: "assets/fonts/iconfont.otf"

  output_class_file: "lib/widgets/iconfont.dart"
  font_name: "Iconfont"
  class_name: "Iconfont"
  format: true

  naming_strategy: "snake"
  normalize: true
  ignore_shapes: false
  recursive: true
  verbose: false
```

`input_svg_dir` and `output_font_file` keys are required.
It's possible to specify any other config file by using `--config-file` option.

## Using API

[svgToOtf][] and [generateFlutterClass][] functions can be used for generating font and Flutter class.

The example of API usage is located in [example folder](example/README.md).

[example folder]: https://github.com/ScerIO/icon_font_generator/tree/master/example/example.dart
[svgToOtf]: https://pub.dev/documentation/icon_font_generator/latest/icon_font_generator/svgToOtf.html
[generateFlutterClass]: https://pub.dev/documentation/icon_font_generator/latest/icon_font_generator/generateFlutterClass.html

## Notes

- Generated OpenType font is using CFF table.
- Generated font is using PostScript Table (post) of version 3.0, i.e., it doesn't contain glyph names.
- Supported SVG elements: path, g, circle, rect, polyline, polygon, line.
- SVG transforms are applied to paths according to specs.
- SVG `<g>` element's children are expanded to the root with transformations applied.
Anything else related to the group is ignored and group referencing is not supported.
- Consider using [Non-zero fill rule][].
- When `ignoreShapes` is set to false,
every SVG shape's (circle, rect, etc.) outline is converted to path.
Note that any attributes like "fill" or "stroke" are ignored and only the outline is used,
so the resulting glyph may look different from SVG icon.
It's recommended to convert every element in SVG to path.
- When `normalize` is set to false, it's recommended that SVG icons have the same height.
Otherwise, final result might not look as expected.
- When Flutter class is generated, static variables names derive from SVG file name
converted to pascal case with non-allowed characters removed.
Name is set to 'unnamed', if it's empty.
Suffix '_{i+1}' is added, if name already exists.

[Non-zero fill rule]: https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/fill-rule

## Contributing

Any suggestions, issues, pull requests are welcomed.

## License

[MIT](https://github.com/ScerIO/icon_font_generator/blob/master/LICENSE)

## Credits

This software is fork  of unsupported package:
* [fontify](https://github.com/westracer/fontify)