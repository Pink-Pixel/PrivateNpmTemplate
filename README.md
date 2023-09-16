# Private Npm Template

Private Npm Template is a template project for creating private NPM packages that you can bring into your own Roblox games. Below you can find instructions on how to generate this yourself, or of course feel free to clone this repo and modify to your own needs.

## Usage
Once cloned or setup, You simply need to run `npm publish` (or `npm publish --registry https://registry-url-here`) to publish your package. You can then install it into your game projects with `npm install @scope/name` (values you should modify within `package.json`).

Once brought into your game project, you can send your modules wherever you like by editing the `*.project.json` that you are using with [Rojo](https://rojo.space). You will need to point to `node_modules` for the `$path` if you are not using any scopes, or points to `node_modules/@scope` if you are. For example, this template assumes a scope of `@client`, and in our main project we might have `tree.ReplicatedStorage.Client.Modules` and within that, set the `$path` to `node_modules/@client/`. This way, any private node modules we create with the `@client` scope, they will all be accessible via `ReplicatedStorage.Client.Modules`.

## Creating from Scratch
First off, this is intended for using private repositories, not publishing to a public registry.

We use [Bytesafe](https://bytesafe.dev). They have a free option which works great for our needs, but if you have an alternative for a private NPM registry then by all means use that. This guide will assume using Bytesafe. Check the features and pricing of [Bytesafe here](https://bytesafe.dev/pricing/) to see what plan you might need and how that might affect you.

### Initial Setup
- [ ] Create your private NPM registry (for example, with [Bytesafe](https://bytesafe.dev))
- [ ] Setup access to your private NPM registry - run `npm --registry https://registry-url-here login` and enter the username, password and email (you can get these from Bytesafe once logged in)
- [ ] Determine various scopes that you wish to use - Personally we use `client`, `shared` and `server` which allows us to place code into different sections in our main project. This is optional and you may not care about scoping
    - [ ] For each scope, set the NPM config to point to your registry by running: `npm config set @scope:registry=https://registry-url-here`. Replace `@scope` with the scope(s) you determined (ie `@client`, `@shared` and `@server`)

### Project Setup
- [ ] Create a GitHub repo
- [ ] Sync GitHub repo to a local folder
- [ ] Open VS Code command palette (`ctrl + shift + P`) and search for `Rojo: Open Menu`
- [ ] Select `Install Now`
- [ ] Select `Create one now` (You can get back here if you cancel out by opening the Rojo menu in command palette again)
- [ ] Add `Packages/`, `node_modules/` and `sourcemap.json` to `.gitignore` file
- [ ] Install Selene - `aftman add Kampfkarren/selene`
- [ ] Configure Selene. At the root, create `selene.toml` and insert: `std = "roblox"`
- [ ] Install StyLua - `aftman add JohnnyMorganz/stylua`
- [ ] Configure StyLua. At the root, create `.vscode/settings.json` and insert:
```json
{
    "[lua]": {
        "editor.defaultFormatter": "JohnnyMorganz.stylua",
        "editor.formatOnSave": true,
        "editor.rulers": [120]
    }
}
```
- [ ] Next, also create `stylua.toml` and insert:
```toml
line_endings = "Windows"
indent_type = "Spaces"
```
- [ ] Delete everything underneath `/src/`
- [ ] Create `src/init.lua` and spin up your module code here. Feel free to use other stuff to build this up like other libs do.
- [ ] Create `tests/init.client.lua` where you can test the integration code
- [ ] Modify `default.project.json` to the following
```json
{
    "name": "PrivateNpmTemplate",
    "tree": {
        "$path": "src"
    }
}
```
- [ ] Create `tests.project.json` and fill it with the following. Adjust it as required
```json
{
    "name": "PrivateNpmTemplate",
    "tree": {
        "$className": "DataModel",

        "ReplicatedStorage": {
            "Client": {
                "$className": "Folder",
                "Modules": {
                    "$className": "Folder",
                    "PrivateNpmTemplate":{
                        "$path": "src/"
                    }
                }
            }
        },

        "StarterPlayer": {
            "StarterPlayerScripts": {
                "Tests": {
                    "$path": "tests/"
                }
            }
        }
    }
}
```
- [ ] Generate correct sourcemap
	- Run `rojo sourcemap .\tests.project.json --output sourcemap.json` if its ever out of sync
- [ ] Initialise NPM `npm init`
	- [ ] Answer all the questions:
		- Package name should be prefixed with the scope (we use:`@client`, `@shared` or `@server`).
		- Version should stay as default (probably)
		- Provide some description.
		- Entry point should be `src/init.lua`.
		- Blank test command
		- git repo should default
		- Give some keywords separated by spaces.
		- Author should be "Email format" ie `Pingu <pingu@email.com>`.
		- License should be `UNLICENSED`.
	- [ ] Remove the `scripts` section of the JSON.
	- [ ] Add the following Json to the end of the `package.json`. This will ensure that only the src is packaged, and that it is published to the correct repository.
```json
  "files": [
    "./src/*",
    "default.project.json"
  ],
  "publishConfig": {
    "registry": "https://repository-url-here"
  }
```
- [ ] To publish to NPM, we can simply run `npm publish`
