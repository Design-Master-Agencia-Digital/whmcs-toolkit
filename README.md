# WHMCS Toolkit for PhpStorm

Completion and typo detection for WHMCS module and hook development in PhpStorm (and IntelliJ IDEA Ultimate with the PHP plugin).

- Hook names inside `add_hook('...')`: all 334 WHMCS hook points, with what each one does.
- Hook variables: keys of `$vars['...']` inside the hook closure, with their types.
- Internal API: the 162 `localAPI('...')` commands and their request parameters (required ones in bold).
- Module settings: keys of the arrays returned by `mymodule_MetaData()` (DisplayName, APIVersion, RequiresServer...) and `mymodule_ConfigOptions()` (FriendlyName, Type, Options, SimpleMode, Loader...), and the six field types (text, password, yesno, dropdown, radio, textarea).
- Payment gateways: `$params` keys in `mygateway_link`, `mygateway_capture`, `mygateway_refund` and `mygateway_storeremote` (invoiceid, amount, returnurl, cardnum, transid, action, gatewayid...) plus the settings of your own `mygateway_config()` (whose keys, FriendlyName/Type System and field types are completed too, and a setting named like a standard parameter such as `currency` or `action` is flagged), and the keys of the array capture, refund and storeremote return (status, transid, rawdata, fee, gatewayid...).
- Missing required parameters: `localAPI('AddClient', [...])` without `email` only fails at runtime. The plugin lists the required parameters missing from the array.
- Quick Documentation: Ctrl+Q (F1 on macOS) or hover on a hook name or API command shows what it does, its `$vars` or parameters (required ones marked) and a link to the official page.
- Typo inspection: a misspelled hook name never fires and WHMCS gives no error. The plugin flags unknown hook names, API commands, misspelled `$vars` keys inside a hook callback, `$params` keys in provisioning functions, and tables and columns in a `Capsule::table()` query, and offers a one-click fix to the closest valid name.
- Live templates: type `whmodule`, `whaddon`, `whhook` or `whapi` and press Tab for a provisioning module, addon module, hook or localAPI call skeleton, named after the file (e.g. `acmehost.php` gives `acmehost_CreateAccount`).

## Getting started

1. In PhpStorm, open Settings > Plugins > Marketplace, search for "WHMCS Toolkit" and install it ([plugin page](https://plugins.jetbrains.com/plugin/35775-whmcs-toolkit)). The 30-day trial starts on first use.
2. Open your WHMCS module or hooks folder. Nothing to configure: the plugin works in any PHP file.
3. Try it:

```php
// includes/hooks/welcome.php: type whhook + Tab, then Ctrl+Space inside the quotes
add_hook('ClientAdd', 1, function (array $vars) {
    // Ctrl+Space inside $vars['...'] lists userid, firstname, email...
    $results = localAPI('SendEmail', [
        'messagename' => 'Client Signup Email',
        'id' => $vars['userid'],
    ]);
});
```

- A typo like `add_hook('ClientAddd', ...)` is underlined; Alt+Enter replaces it with `ClientAdd`.
- In `modules/servers/acmehost/acmehost.php`, type `whmodule` + Tab for a full provisioning module named `acmehost_*`.

Paid plugin with a 30-day free trial, sold through JetBrains Marketplace.

Support and bug reports: open an issue in this repository or write to contato@designmaster.com.br.

License: [EULA](EULA.md). Privacy: [the plugin collects no data](PRIVACY.md).

Made by [Design Master](https://www.designmaster.com.br/?utm_source=github.com&utm_medium=referral&utm_campaign=whmcs-toolkit), a WHMCS hosting company in Brazil.
