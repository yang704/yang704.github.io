# Typescript环境配置(简单配置)

## VS Code配置插件

- `TypeScript Importer`：这一插件会收集你项目内所有的类型定义，在你敲出:时提供这些类型来进行补全。如果你选择了一个，它还会自动帮你把这个类型导入进来。
- `Move TS`:这一插件在重构以及像我们这样写 demo 的场景下很有帮助。它可以让你通过编辑文件的路径，直接修改项目的目录结构。
- `ErrorLens`:这一插件能够把你的 VS Code 底部问题栏的错误下直接显示到代码文件中的对应位置。

## 安装TypeScript

- 全局安装TS
	```
	npm install -g typescript
	```
- 全局安装ts-node
	```
	npm install -g ts-node
	```
- 全局安装ts-node-dev
	```
	npm i ts-node-dev -g
	```
- 生成TypeScript的项目配置文件（tsconfig.json）
	```
	//未全局安装typescript
	npx --package typescript tsc --init
	
	//已全局安装typescript的情况下
	tsc --init
	```
- 运行ts文件
	```
	ts-node index.ts
	//或者
	ts-node-dev --respawn --transpile-only app.ts
	```