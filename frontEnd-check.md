# 代码检查-ESLint

## ts

```bash
npm install -g eslint
```
由于 ESLint 默认使用 Espree 进行语法解析:
```bash
npm install -g typescript @typescript-eslint/parser
```

安装对应的插件 @typescript-eslint/eslint-plugin 它作为 eslint 默认规则的补充，提供了一些额外的适用于 ts 语法的规则。
```bash
npm install -g @typescript-eslint/eslint-plugin
```

ESLint 的时候检查一个文件的时候，它会首先尝试读取该文件的目录下的配置文件，然后再一级一级往上查找，将所找到的配置合并起来，作为当前被检查文件的配置。