# TS config

```jsonc
{
  "compilerOptions": {
    "rootDir": ".",
    // specifies base directory for resolving non-relative module names
    "baseUrl": ".",
    // specifies a mapping of module names to paths
    "paths": {
      "@/shared/*": ["shared/*"]
    },
    // output build directory for compiled code
    "outdir": "lib",
    // ECMAScript version code for compiled code
    "target": "ESNext",
    // specify set of bundled library declaration files that
    // describe the target runtime environment
    "lib": ["DOM", "ESNext"],
    // enables incremental compilaton, which allows compiler to reuse
    // results of a previous compilation to speed up subsequent compilations
    "incremental": true,
    "declaration": true,
    // specifies whether to emit output files
    "noEmit": true,
    "experimentalDecorators": true,
    "forceConsistentCasingInFileNames": true,
    "esModuleInterop": true,
    // sets the module system code for compiled code
    // when transpiling with tsc, use module: NodeNext, moduleResolution: NodeNext
    // for everything else, use module: ESNext, moduleResolution: Bundler
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noImplicitThis": false,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "strict": true,
    "strictPropertyInitialization": false,
    "strictFunctionTypes": false,
    "removeComments": true,
    // specifies whether to skip type checking of declaration (.d.ts) files
    "skipLibCheck": true,
    "useUnknownInCatchVariables": true
    // "allowJs": true,
    // "checkJs": false,
  }
}
```
