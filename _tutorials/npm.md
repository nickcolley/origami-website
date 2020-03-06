---
title: Origami via npm
description: A guide on how to use Origami via npm.
cta: Learn how to Origami via npm
---

# {{page.title}}


If migrating from Bower to npm, [please refer to our migration guide](https://origami.ft.com/docs/tutorials/bower-to-npm/).

## Benefits

- Tooling such as Dependabot/Renovate/Greenkeeper/Snyk will work for the Origami dependencies you have.

- Other tooling which by default only work with npm dependencies will now work without requiring any changes. I.E. Webpack.

## Drawbacks

- The Origami Team have not finalised how Origami components on npm should work, it is considered not stable.

- Need to use extra tooling to ensure multiple versions of Origami are not included in the dependency tree.

- Documentation for Origami assumes the Bower interface and not the npm interface. I.E. The package-names don't start with `@financial-times`.


## How to avoid multiple versions of Origami in the dependency tree

There is a tool called [lockspot](https://www.npmjs.com/package/lockspot) which can be used to report about any packages which are included with more than one version, such as Origami dependencies. E.G. Detecting that `o-table@6` and `o-table@7` are both in the dependency tree.

Below we will set up [lockspot](https://www.npmjs.com/package/lockspot) to fail a project which has multiple versions of dependencies from `@financial-times`.

Here is how to use it with an npm/yarn/pnpm project:

1. Create a file called `./scripts/lockspot` and make it executable.

``` bash
> mkdir ./scripts
> touch ./scripts/lockspot
> chmod +x ./scripts/lockspot
```

2. Place this into the `./scripts/lockspot` file.

```bash
#!/bin/bash
set -e
PACKAGE_LOCK_FILE="package-lock.json"

if [ ! -f "$PACKAGE_LOCK_FILE" ]; then
	echo "Could not find $PACKAGE_LOCK_FILE when running ./scripts/lockspot"
	exit 1
fi

# Verify that financial-times namespaced dependencies are only included once - this
# ensures that client builds don't end up including multiple versions of Origami
# components, bloating code size unnecessarily.
if
	[ -f "$PACKAGE_LOCK_FILE" ] &&
	! npx lockspot flat --pattern '^@financial-times/' --file "$PACKAGE_LOCK_FILE";
then
	echo -e "\033[33mChecking the package-lock.json file showed some FT components were included multiple times with different versions. This is currently not supported by Origami components. This can also result in client builds including multiple copies of the CSS/JS for those packages, increasing build size. Please check the dependency ranges to ensure each dependency is only included once.\n\nFT components included more than once, with their count:"
	npx lockspot depcount --min 2 --pattern '^@financial-times/' --file "$PACKAGE_LOCK_FILE"
	echo -e "\033[0m"
	exit 1
fi
```

3. Run the `./scripts/lockspot` executable before `npm pack`/`npm publish`/`npm install` by adding a `prepare` script to the `package.json`.

``` json
{
    "prepare": "./scripts/lockspot"
}
```


## How to get Origami Sass to work

Currently Origami components on npm have the exact same Sass as their Bower counterparts. To get the Sass compiler to work we will need to configure the `includePaths` of the compiler to look in `node_modules` and `node_modules/@financial-times`.

This can usually be done via the `.sassrc` file, like below:
```json
{
	"includePaths": [
        "node_modules",
        "node_modules/@financial-times"
    ]
}
```