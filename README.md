# issue-npm-publish-with-workspaces

This repository illustrate an issue that can be faced with `npm` when trying to publish a package within a workspace.

*Original issue: https://github.com/npm/cli/issues/7726*

### Info

- **platform**: macOS Monterey 12.7.5
- **node**: v20.15.1
- **npm**: 10.8.2

### Issue

Considering the following mono-repo workspace:

```sh
my-project
├── dist
│   └── a
│       └── src
│       │    ├── file.d.ts
│       │    └── file.js
│       └── package.json # { "name: "a", "version": "1.0.0" }
├── projects
│   └── a
│       └── src
│       │    └── file.ts
│       └── package.json # { "name: "a", "version": "1.0.0" }
└── package.json # { "name: "my-project", "workspaces": [ "projects/a" ] }
```

When releasing a new project the idea is to:
1. Set the working-directory to the desired project (ex: `cd projects/a`)
2. Build the project into the root `dist` folder (ex: `npm run build`)
3. Publish the project (ex: `npm run release` which runs `npm publish ../../dist/a --access=public`)

When specifying `workspaces` in the root package.json, the step 3 does not work as expected.

It ignores the `<package-spec>` parameter completely and publish the files in the current directory instead:
```sh
$ npm run release

> a@1.0.0 publish
> npm publish ../../dist/a --dry-run

npm notice
npm notice 📦  a@1.0.0
npm notice Tarball Contents
npm notice 172B package.json
npm notice 0B src/file.ts
npm notice Tarball Details
npm notice name: a
npm notice version: 1.0.0
npm notice filename: a-1.0.0.tgz
...
```

The `<package-spec>` parameter has no effect at all. Even a wrong path will not display an error and publish the current directory instead (ex: `npm publish ./unknown-dir --dry-run`).

### Note

The `pack` command on the contrary is working as expected and ignores the workspace:

```sh
$ npm pack ../../dist/a --dry-run

npm warn Ignoring workspaces for specified package(s)
npm notice
npm notice 📦  a@1.0.0
npm notice Tarball Contents
npm notice 44B package.json
npm notice 0B src/file.d.ts
npm notice 0B src/file.js
npm notice Tarball Details
npm notice name: a
npm notice version: 1.0.0
npm notice filename: a-1.0.0.tgz
...
```

### Related

I had opened this [issue](https://github.com/npm/cli/issues/5745), 2 years ago, that is quite similar, because `npm publish <package-spec>` was also not working as expected.


### How to reproduce

1. `git clone https://github.com/Badisi/issue-npm-publish-with-workspaces-2`
2. `npm install`
3. `cd projects/a`
4. `npm run release`
