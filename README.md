# pnpm Not Resolution Skipping Issue Reproduction Repo

This repository reproduces an issue [pnpm/pnpm#9887](https://github.com/pnpm/pnpm/issues/9887) where pnpm fails to skip the resolution step when using workspace aliases.

## Problem Description

When using workspace dependencies with aliased names and specific version ranges (e.g., `"aliased": "workspace:@private/foo@*"`), pnpm correctly resolves and links the packages but **does not skip the resolution step** on subsequent `pnpm install` runs, even when the lockfile is up to date.

This results in:

- Slower installation times in large repositories
- Unnecessary lockfile changes on every install
- Missing "Lockfile is up to date, resolution step is skipped" message

## Expected Behavior

pnpm should skip the resolution step when the lockfile is up to date, similar to how it behaves with standard workspace dependencies like `"@private/foo": "workspace:*"`.

## Current Setup

### Project Structure

```plaintext
├── package.json (root)
├── packages/
│   └── foo/
│       └── package.json (@private/foo)
└── pnpm-workspace.yaml
```

### Root package.json

```json
{
  "dependencies": {
    "aliased": "workspace:@private/foo@*"
  }
}
```

### Workspace package

```json
{
  "name": "@private/foo",
  "private": true,
  "version": "0.1.0"
}
```

## Reproduction Steps

1. Clone this repository
2. Run `pnpm install` (first time - resolution occurs)
3. Run `pnpm install` again (second time - resolution occurs again, should be skipped)

### Expected Result (with standard workspace deps)

```bash
$ pnpm install
Lockfile is up to date, resolution step is skipped
```

### Actual Result (with aliased workspace deps)

```bash
$ pnpm install
# Resolution step runs every time
```

## Workarounds

The issue can be avoided by using either of these approaches:

1. **Remove the aliased dependency entirely**
2. **Use standard workspace syntax**: `"@private/foo": "workspace:*"`

Both workarounds result in proper resolution skipping behavior.

## Environment

- **Node.js**: v22.15.0
- **pnpm**: v10.15.0
- **OS**: macOS 26.0 Beta (25A5346a)

## Related Context

This approach was inspired by [@pnpm/core's self-reference pattern](https://github.com/pnpm/pnpm/blob/05dd45ea82fff9c0b687cdc8f478a1027077d343/pkg-manager/core/package.json#L128) in the pnpm repository, where `@pnpm/core` declares itself as a devDependency. This pattern offers better compatibility than using `tsconfig.json#paths` or `package.json#imports`.
