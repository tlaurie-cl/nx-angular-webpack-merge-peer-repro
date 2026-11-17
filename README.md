# @nx/angular: `webpack-merge` optional peer range (`^5.8.0`) rejects webpack-merge 6

Minimal reproduction for an `npm install` failure after upgrading `@nx/angular` from **23.1.1** to **23.3.x** (and still present in **24.0.6** and the current canary).

## Background

[nrwl/nx#36310](https://github.com/nrwl/nx/pull/36310) moved `webpack-merge` in `@nx/angular` from `dependencies` to an **optional peer dependency**. The range stayed the same:

| `@nx/angular` | where `webpack-merge` is declared | range |
| --- | --- | --- |
| 23.1.1 | `dependencies` | `^5.8.0` |
| 23.3.2 | `peerDependencies` (optional) | `^5.8.0` |
| 24.0.6 (latest) | `peerDependencies` (optional) | `^5.8.0` |

As a regular dependency, `^5.8.0` was harmless. npm just installed a nested copy of webpack-merge 5 for `@nx/angular`, and the workspace kept its own version. As a peer dependency, the workspace's copy has to match the range. Any workspace already on **webpack-merge 6.x** (6.0.0 was released 2024-07-02) now fails to install.

## Steps to reproduce

```sh
git clone https://github.com/tlaurie-cl/nx-angular-webpack-merge-peer-repro
cd nx-angular-webpack-merge-peer-repro
npm install
```

`package.json` contains only:

```json
"devDependencies": {
  "@nx/angular": "23.3.2",
  "webpack-merge": "^6.0.1"
}
```

## Actual result

```
npm error code ERESOLVE
npm error ERESOLVE could not resolve
npm error
npm error While resolving: @nx/angular@23.3.2
npm error Found: webpack-merge@6.0.1
npm error node_modules/webpack-merge
npm error   dev webpack-merge@"^6.0.1" from the root project
npm error
npm error Could not resolve dependency:
npm error peerOptional webpack-merge@"^5.8.0" from @nx/angular@23.3.2
npm error node_modules/@nx/angular
npm error   dev @nx/angular@"23.3.2" from the root project
npm error
npm error Conflicting peer dependency: webpack-merge@5.10.0
npm error node_modules/webpack-merge
npm error   peerOptional webpack-merge@"^5.8.0" from @nx/angular@23.3.2
npm error   node_modules/@nx/angular
npm error     dev @nx/angular@"23.3.2" from the root project
npm error
npm error Fix the upstream dependency conflict, or retry this command with --force or --legacy-peer-deps to accept an incorrect (and potentially broken) dependency resolution.
```

Changing `@nx/angular` to `23.1.1` installs without errors.

## Expected result

`npm install` succeeds. The optional peer range should accept webpack-merge 6, e.g. `^5.8.0 || ^6.0.0`.

## Notes

- `@nx/angular` uses `merge` from webpack-merge: the named export in `dist/src/builders/utilities/webpack.js`, and the default export in the config that `convert-to-rspack` generates. webpack-merge 6.0.1 still provides both (`export { ..., merge, merge as default, ... }`). It now requires `node >= 18`, which Nx already requires.
- `webpackMergeVersion` in `packages/angular/src/utils/versions.ts` is also `^5.8.0`. The `update-23-1-0/add-optional-webpack-packages` migration and the `setup-mf` generator use it to add webpack-merge to workspaces, so they may need the same bump.
- The only workarounds are `--legacy-peer-deps` / `--force`, or downgrading webpack-merge to 5.x.

## Environment

- node v26.12.1 / npm 11.20.0 (also fails in CI on node 22, see the Actions tab)
- `@nx/angular` 23.3.2 (also 24.0.6, 24.1.0-beta.1)
- `webpack-merge` 6.0.1
