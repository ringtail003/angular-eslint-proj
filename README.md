## 目的

Angular の Linter を ESLint & Prettier にする。

- node 14.8
- Angular 11

## Angular プロジェクトの作成

```sh
$ npx -p @angular/cli ng new angular-eslint-proj

# プロジェクト作成時のオプションはやりたい事の文脈と関連しないので適当に選択しておく

? Do you want to enforce stricter type checking and stricter bundle budgets in the workspace?
  This setting helps improve maintainability and catch bugs ahead of time.
  For more information, see https://angular.io/strict Yes
? Would you like to add Angular routing? Yes
? Which stylesheet format would you like to use? CSS

$ cd angular-eslint-proj
```

## TSLint > ESLint へのマイグレーション

> 参考ドキュメント  
> [angular-eslint:
 Migrating an Angular CLI project from Codelyzer and TSLint](https://github.com/angular-eslint/angular-eslint#migrating-an-angular-cli-project-from-codelyzer-and-tslint)

Angular プロジェクトに ESLint を組み込む。

```sh
$ ng add @angular-eslint/schematics

Installing packages for tooling via npm.
Installed packages for tooling via npm.
UPDATE package.json (1704 bytes)
✔ Packages installed successfully.
```

Linter を TSLint から ESLint に変更する。

```sh
$ ng g @angular-eslint/schematics:convert-tslint-to-eslint angular-eslint-proj

CREATE .eslintrc.json (1084 bytes)
UPDATE angular.json (3606 bytes)
```

`.eslintrc.json` が生成され `angular.json` に変更が加えられる。

```js
// .eslintrc.json
{
  "root": true,
  "ignorePatterns": [
    "projects/**/*"
  ],
  "overrides": [
    {
      "files": [
        "*.ts"
      ],
      "parserOptions": {
        "project": [
          "tsconfig.json",
          "e2e/tsconfig.json"
        ],
        "createDefaultProgram": true
      },
      "extends": [
        "plugin:@angular-eslint/ng-cli-compat",
        "plugin:@angular-eslint/ng-cli-compat--formatting-add-on",
        "plugin:@angular-eslint/template/process-inline-templates",
      ]
    }
  ]
  ...
}
```

```js
// angular.json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "angular-eslint-proj": {
      "architect": {
        "lint": {
          "builder": "@angular-eslint/builder:lint",
          "options": {
            "lintFilePatterns": [
              "src/**/*.ts",
              "src/**/*.html"
            ]
          }
        }
      }
    }
  }
  ...
}
```

## TSLint の削除

```sh
$ rm ./tslint.json

# Codelyzer（TSLint のルールセット）も使わないので削除
$ npm uninstall codelyzer
```

## Prettier 導入

> 参考ドキュメント  
> [Prettier: Basic Configuration](https://prettier.io/docs/en/configuration.html#basic-configuration)

Prettier の設定ファイルを作成する。

```sh
$ touch .prettierrc.js
```

```js
// .prettierrc.js
module.exports = {};
```

Prettier の設定を ESLint のルールとして取り込む。
これによって Prettier が整形したコードが Lint のエラーとして検出される事を防ぐ。

```sh
$ npm install -D eslint-config-prettier
$ npm install -D prettier
```

```js
// .eslintrc.json
"extends": [
  "plugin:@angular-eslint/ng-cli-compat",
  "plugin:@angular-eslint/ng-cli-compat--formatting-add-on",
  "plugin:@angular-eslint/template/process-inline-templates",
  
  // この 2 行を追加
  "prettier",
  "prettier/@typescript-eslint"
],
```

ESLint と Prettier の設定にコンフリクトがないかチェックする。
コンフリクトする場合は `@angular-eslint/xxx` を無効にする。

> 参考ドキュメント  
> [eslint-config-prettier: CLI helper tool](https://github.com/prettier/eslint-config-prettier#cli-helper-tool)

```sh
./node_modules/.bin/eslint-config-prettier ./src/app/app.component.ts

# コンフリクトなし
No rules that are unnecessary or conflict with Prettier were found.
```

## Prettier のルールを追加

```js
// .prettierrc.js
module.exports = {
  // string のリテラルにシングルクォートを強制
  singleQuote: false
};
```

利用できるルールは公式ドキュメントに記載されている。

* [Prettier: Options](https://prettier.io/docs/en/options.html)
* [Prettier: Configuration Schema](https://prettier.io/docs/en/configuration.html#configuration-schema)

## @typescript-eslint のルールを追加

error/warn/off のいずれかを指定する。

```js
"overrides": {
  "rules": {
    // 空のインターフェースを許容しない
    "@typescript-eslint/no-empty-interface": "error"
  }
}
```

利用できるルールは公式ドキュメントに記載されている。

* [npm: @typescript-eslint/eslint-plugin](https://www.npmjs.com/package/@typescript-eslint/eslint-plugin)

## WIP: ESLint のルールを追加

利用できるルールは公式ドキュメントに記載されている。

> [ESLint: Rules](https://eslint.org/docs/rules/)

## Lint の実行

`ng lint` でエラーを検出する。`ng lint --fix` でエラーが検出されたコードを修正する。

```sh
> ng lint

/angular-eslint-proj/src/app/app.component.ts

error  An empty interface is equivalent to `{}`  @typescript-eslint/no-empty-interface

✖ 1 problem (1 error, 0 warnings)
Lint errors found in the listed files.
```

## ESLint の推奨ルールを取り込む


```js
// .eslintrc.json
{
  ...
  "overrides": [
    {
      "files": [],
      "parserOptions": {},
      "extends": [
        // この行を追加
        "eslint:recommended",
        
        "plugin:@angular-eslint/ng-cli-compat",
        "plugin:@angular-eslint/ng-cli-compat--formatting-add-on",
        "plugin:@angular-eslint/template/process-inline-templates",
      ]
    }
  ]
  ...
}
```

Lint を実行するとエラーが大量に検出される。

```sh
$ ng lint

Linting "angular-eslint-proj"...

/Users/matsuoka/work/ringtail003/angular-eslint-proj/src/app/app.component.spec.ts
   5:1  error  'describe' is not defined    no-undef
   6:3  error  'beforeEach' is not defined  no-undef
  17:3  error  'it' is not defined          no-undef
  ...

✖ 8 problems (8 errors, 0 warnings)

/Users/matsuoka/work/ringtail003/angular-eslint-proj/src/app/app.component.ts
  12:11  error  'x' is assigned a value but never used  no-unused-vars

✖ 1 problem (1 error, 0 warnings)

/Users/matsuoka/work/ringtail003/angular-eslint-proj/src/test.ts
  11:11  error  'path' is defined but never used    no-unused-vars
  11:25  error  'deep' is defined but never used    no-unused-vars
  ...
```

不必要なルールは ESLint のルールから除外する。

```js
{
  "overrides": [
    {
      "rules": {
        "@angular-eslint/component-selector": [],
        "@angular-eslint/directive-selector": [],
        
        // 除外する
        "no-undef": "off",
        "no-unused-vars": "off"
      }
    }
  ]
  ...
}
```

## VSCode の設定

VSCode のエラー表示（赤い破線）を ESLint と連動させる。

* ADD: [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
* ADD: [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
* REMOVE: [TSLint](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-tslint-plugin)

デフォルトのフォーマッターを Prettier に設定し、ファイル保存時にフォーマットを実行する。

```js
"[typescript]": {
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode"
},
"editor.codeActionsOnSave": {
  "source.fixAll.eslint": true
}
```
