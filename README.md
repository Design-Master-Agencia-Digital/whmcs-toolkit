# WHMCS Toolkit for PhpStorm

Completion and typo detection for WHMCS module and hook development in PhpStorm (and IntelliJ IDEA Ultimate with the PHP plugin).

- Hook names inside `add_hook('...')`: all 334 WHMCS hook points, with what each one does.
- Hook variables: keys of `$vars['...']` inside the hook closure, with their types.
- Internal API: the 162 `localAPI('...')` commands and their request parameters (required ones in bold).
- Missing required parameters: `localAPI('AddClient', [...])` without `email` only fails at runtime. The plugin lists the required parameters missing from the array.
- Quick Documentation: Ctrl+Q (F1 on macOS) or hover on a hook name or API command shows what it does, its `$vars` or parameters (required ones marked) and a link to the official page.
- Typo inspection: a misspelled hook name never fires and WHMCS gives no error. The plugin flags unknown hook names and API commands and offers a one-click fix to the closest valid name.
- Live templates: type `whmodule`, `whaddon`, `whhook` or `whapi` and press Tab for a provisioning module, addon module, hook or localAPI call skeleton, named after the file (e.g. `acmehost.php` gives `acmehost_CreateAccount`).

Paid plugin with a 30-day free trial, sold through JetBrains Marketplace.

Support and bug reports: open an issue in this repository or write to contato@designmaster.com.br.

License: [EULA](EULA.md).

Made by [Design Master](https://www.designmaster.com.br/?utm_source=github.com&utm_medium=referral&utm_campaign=whmcs-toolkit), a WHMCS hosting company in Brazil.
