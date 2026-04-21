
This is a repro for https://github.com/webpack/webpack/issues/20671


## Symptoms

Running a TypeScript build fails because some types are colliding.

```bash
❯ npm run build
npm warn Unknown user config "always-auth". This will stop working in the next major version of npm.

> build
> tsc

node_modules/@types/eslint-scope/index.d.ts:7:5 - error TS2416: Property 'scopes' in type 'ScopeManager' is not assignable to the same property in base type 'ScopeManager'.
  Type 'import("/Users/sgoetz/workspace/github.com/eslint-webpack-repro/node_modules/@types/eslint-scope/index").Scope[]' is not assignable to type 'import("/Users/sgoetz/workspace/github.com/eslint-webpack-repro/node_modules/eslint/lib/types/index").Scope.Scope[]'.
    Type 'import("/Users/sgoetz/workspace/github.com/eslint-webpack-repro/node_modules/@types/eslint-scope/index").Scope' is not assignable to type 'import("/Users/sgoetz/workspace/github.com/eslint-webpack-repro/node_modules/eslint/lib/types/index").Scope.Scope'.
      Types of property 'type' are incompatible.
        Type '"function" | "with" | "block" | "module" | "switch" | "global" | "class" | "catch" | "for" | "function-expression-name" | "TDZ"' is not assignable to type '"function" | "with" | "block" | "module" | "switch" | "global" | "class" | "catch" | "for" | "function-expression-name" | "class-field-initializer" | "class-static-block"'.
          Type '"TDZ"' is not assignable to type '"function" | "with" | "block" | "module" | "switch" | "global" | "class" | "catch" | "for" | "function-expression-name" | "class-field-initializer" | "class-static-block"'.

7     scopes: Scope[];

...

Found 17 errors in the same file, starting at: node_modules/@types/eslint-scope/index.d.ts:7
```

## Root cause

Webpack imports `@types/eslint-scope` which imports types from `eslint`.
But the version of `eslint-scope` the types are for is outdated and not compabtible with ESLint 10

```
❯ npm ls @types/eslint-scope
npm warn Unknown user config "always-auth". This will stop working in the next major version of npm.
eslint-webpack-repro@ /Users/sgoetz/workspace/github.com/eslint-webpack-repro
└─┬ webpack@5.106.2
  └── @types/eslint-scope@3.7.7
```

## Possible fixes

### Remove `@types/eslint-scope` (fix)

This package is officially deprecated and should not be used anymore. These types are part of `eslint-scope` directly.

### Use `"types": []` in `tsconfig.json` (fix)

This is a clean approach and is going to be the default starting with TypeScript 6.
This ensures that only the explicitly defined types are imported by `tsc`

### Use `"skipLibCheck": true` in `tsconfig.json` (workaround)

https://www.typescriptlang.org/tsconfig/#skipLibCheck

This looks like a workaround that disables an important feature of the TypeScript compiler but would hide the error.

### Stay on ESLint 9 (workaround)

The types are broken only since ESLint 10, this is issue does not occur in ESLint 9 and below.
