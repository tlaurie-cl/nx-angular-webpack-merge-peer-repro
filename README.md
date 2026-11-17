# @nx/angular: `webpack-merge` optional peer range (`^5.8.0`) rejects webpack-merge 6

Minimal reproduction for [nrwl/nx#37925](https://github.com/nrwl/nx/issues/37925): `npm install` fails after upgrading `@nx/angular` from **23.1.1** to **23.3.x** (still present in **24.0.6** and the current canary).

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

---

This repo was created with the aid of Claude Opus 5.5 High.
