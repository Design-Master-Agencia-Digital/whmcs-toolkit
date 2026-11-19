# Live templates

WHMCS Toolkit ships 56 live templates. Open a `.php` file, type the abbreviation, press Tab. Nothing to configure. In PhpStorm: Settings > Editor > Live Templates > WHMCS lists them all and lets you edit the body.

## Example: do something when an invoice is paid

1. Create `includes/hooks/paid.php` in your WHMCS folder.
2. Type `whinvoicepaid` and press Tab. You get this, with the cursor on the empty line:

```php
add_hook('InvoicePaid', 1, function (array $vars) {
    $invoice = localAPI('GetInvoice', ['invoiceid' => (int) $vars['invoiceid']]);
    if ($invoice['result'] !== 'success') {
        logActivity('InvoicePaid hook: GetInvoice failed: ' . $invoice['message']);
        return;
    }
    // your code here
});
```

3. Adapt it: `$invoice` already holds the full invoice (total, status, items, clientid). To email the client, call `localAPI('SendEmail', ['messagename' => 'Invoice Payment Confirmation', 'id' => $vars['invoiceid']])` below the check. The `logActivity` line shows up in Utilities > Logs > Activity Log if the invoice cannot be loaded, so a failure never stops the other hooks.
4. Ctrl+Space inside `$vars['...']` lists the keys WHMCS passes to this hook, and Ctrl+Q on `GetInvoice` shows its parameters.

## Example: a provisioning module in one minute

1. Create `modules/servers/acmehost/acmehost.php`.
2. Type `whmodule` and press Tab. The file name fills the module name, so you get `acmehost_MetaData`, `acmehost_ConfigOptions`, `acmehost_CreateAccount`, `acmehost_SuspendAccount`, `acmehost_UnsuspendAccount`, `acmehost_TerminateAccount`, `acmehost_ChangePassword` and `acmehost_ChangePackage`. Type the display name once (the first stop) and the cursor lands inside `acmehost_CreateAccount`.
3. Adapt it: the template declares one `ConfigOptions` field, `Package`. WHMCS hands it to every function as `$params['configoption1']`, so the call to your server is a couple of lines inside the `try`:

```php
$api = new AcmeApi($params['serverhostname'], $params['serverpassword']);
$api->create($params['username'], $params['domain'], $params['configoption1']);
```

   Ctrl+Space inside `$params['...']` lists every key WHMCS passes here (serviceid, domain, username, serverip, configoption1-24...). A typo such as `$params['serverhostnme']` is underlined and Alt+Enter fixes it.
4. Anything your code throws is caught, written with `logModuleCall` and returned as the error text WHMCS shows to the admin. The request and response appear under Utilities > Logs > Module Log (turn on Enable Debug Logging there first). Returning `success` marks the action as done.
5. Fill in the other functions the same way, and delete the ones your server does not support.

## Example: a payment gateway in two files

1. Create `modules/gateways/acmepay.php` and type `whgateway` + Tab. The file name fills the module name: you get `acmepay_MetaData`, `acmepay_config` and `acmepay_link`. Type the display name once (the first stop).
2. Adapt `acmepay_link`: WHMCS calls it when the client sees the invoice. Build the payment button from `$params`, for example `$params['invoiceid']`, `$params['amount']`, `$params['currency']`, `$params['systemurl']` and `$params['returnurl']`. Ctrl+Space inside `$params['...']` lists what WHMCS passes here, and an unknown key is underlined.
3. Create `modules/gateways/callback/acmepay.php` and type `whcallback` + Tab. It loads WHMCS, checks that the gateway is active, validates the invoice (`checkCbInvoiceID`) and the transaction id (`checkCbTransID`, rejects duplicates), logs the call to Billing > Gateway Log and marks the invoice paid with `addInvoicePayment`.
4. Adapt it: replace `$_POST['invoiceid']`, `$_POST['transid']` and `$_POST['amount']` with the fields your provider sends, and verify the provider's signature before the `checkCb*` calls. Point the provider's webhook to `https://your-whmcs/modules/gateways/callback/acmepay.php`.

## Example: show a custom value in the client area

1. Create `includes/hooks/clientarea.php` and type `whclientarea` + Tab. You get a `ClientAreaPage` hook that returns an array, with the cursor inside it.
2. Adapt it: every key you return becomes a Smarty variable on the client area pages.

```php
add_hook('ClientAreaPage', 1, function (array $vars) {
    return ['supportHours' => 'Mon-Fri 9:00-18:00'];
});
```

3. In your template (for example `templates/<theme>/footer.tpl`) print it with `{$supportHours}`. Reload any client area page and the text appears.
4. Type `$vars['` inside the callback and Ctrl+Space lists the keys WHMCS passes to this hook.

## Example: catch a typo before WHMCS silently ignores it

WHMCS does not warn about a misspelled hook name or `$vars` key: the hook never fires, or reads `null`. The WHMCS inspection underlines it while you type.

1. In the `whinvoicepaid` hook above, change `$vars['invoiceid']` to `$vars['invoicid']`.
2. The key is underlined as unknown for `InvoicePaid`. Press Alt+Enter and pick the suggested fix: it is replaced with the closest valid key.
3. The same applies to hook names (`add_hook('InvoicePad', ...)`), API commands (`localAPI('GetInvoce', ...)`), table and column names in `Capsule::table()` and `$params['...']` keys in modules.

## Example: query the WHMCS database without guessing column names

WHMCS table and column names are easy to misremember (`tblhosting.domainstatus`, `tblinvoices.datepaid`), and a wrong column only fails at runtime. The plugin knows the 171 core tables of WHMCS 9.0.

1. In any hook or module file, add `use WHMCS\Database\Capsule;` and type `Capsule::table('tblinv` then Ctrl+Space: `tblinvoices` (22 columns) is listed. Ctrl+Q (F1 on macOS) on the table name lists every column and its type.
2. Chain `->where('` and press Ctrl+Space: the columns of `tblinvoices` are offered (`status`, `total`, `duedate`, `datepaid`...). Complete the query:

```php
$overdue = Capsule::table('tblinvoices')
    ->where('status', 'Unpaid')
    ->where('duedate', '<', date('Y-m-d'))
    ->orderBy('duedate')
    ->pluck('total', 'id');
```

3. Change `duedate` to `duedat`: it is underlined as an unknown column of `tblinvoices` and Alt+Enter replaces it with `duedate`. The same works for `select`, `value`, `pluck`, `orderBy` and the keys of `insert`/`update`, and after a `join('tblclients', ...)` the columns of both tables are checked.

## Example: an addon module with an admin page

1. Create `modules/addons/acmenotes/acmenotes.php`.
2. Type `whaddon` and press Tab. The file name fills the module name, so you get `acmenotes_config`, `acmenotes_activate`, `acmenotes_deactivate` and `acmenotes_output`. Type the display name once and the cursor lands inside `acmenotes_output`.
3. Adapt it: `acmenotes_output` prints the page admins see under Addons. WHMCS passes `$vars` with your module settings and `$vars['modulelink']`, the URL of this page for links and forms:

```php
function acmenotes_output($vars)
{
    echo '<a href="' . $vars['modulelink'] . '&amp;action=hello">Say hello</a>';
}
```

4. To store data, create the table in `acmenotes_activate` with `\WHMCS\Database\Capsule::schema()->create(...)` and return `['status' => 'success', 'description' => '...']` (use `'status' => 'error'` on failure). In a `Capsule::table()` query the plugin completes table and column names and underlines a typo.
5. Turn it on in Setup > Addon Modules: Activate, then Configure to give an admin role access. The template leaves `fields` empty; add your settings there and read them back from `$vars`.

## Example: a domain registrar module in one minute

1. Create `modules/registrars/acmedomains/acmedomains.php` and type `whregistrar` + Tab. The file name fills the module name: you get `acmedomains_MetaData`, `acmedomains_getConfigArray`, `acmedomains_RegisterDomain`, `acmedomains_GetNameservers` and `acmedomains_SaveNameservers`. Type the display name once (the first stop).
2. Adapt `getConfigArray`: each key becomes a field under Configuration > System Settings > Domain Registrars (here `APIKey`). WHMCS hands the saved values back in `$params`, so the key is read as `$params['APIKey']`.
3. Write the registration in the `try` block, where the cursor stops. WHMCS calls `RegisterDomain` with the domain parts and contact data in `$params`, for example `$params['sld']` (`example`), `$params['tld']` (`com`), `$params['regperiod']` (years), `$params['ns1']` and `$params['domainid']`. Call your registrar's API there, then leave `return ['success' => true];` as is.
4. Report failures with the `catch` block: `['error' => $e->getMessage()]` shows the message in the admin area instead of marking the domain registered.
5. Fill `GetNameservers` (return `ns1` to `ns5` as the registrar has them) and `SaveNameservers` (read `$params['ns1']`...). Activate the module under Domain Registrars and assign it to a TLD in Domain Pricing.

## Example: block an order at checkout

1. Create `includes/hooks/checkout.php` and type `whcheckout`, then Tab. The cursor stops inside the `if`:

```php
add_hook('ShoppingCartValidateCheckout', 1, function (array $vars) {
    $errors = [];
    if (empty($vars['companyname'])) {
        $errors[] = 'Please enter your company name to place an order';
    }
    return $errors;
});
```

2. Replace the condition with your rule. `$vars` carries the checkout form: `companyname`, `promocode`, `notes`, `paymentmethod`, `custtype` (`new` or `existing`) and the new-client fields. Ctrl+Space inside `$vars['...']` lists them, and a misspelled key is underlined (see the typo example above).
3. Every string you put in `$errors` is shown to the client and no order or invoice is created. Return an empty array to let the checkout continue.

## Example: a daily job that never breaks the cron

1. Create `includes/hooks/daily.php`, type `whcron` and press Tab. The body sits inside a `try` block, so one failing task does not stop the other `DailyCronJob` hooks:

```php
add_hook('DailyCronJob', 1, function (array $vars) {
    try {
        // your code here
    } catch (\Throwable $e) {
        logActivity('DailyCronJob hook failed: ' . $e->getMessage());
    }
});
```

2. Call WHMCS from it with `whapi`: type it, Tab, fill the command (for example `GetInvoices`) and the array. The generated check logs `GetInvoices failed: ...` to the Activity Log when the result is not `success`.
3. Check the outcome in Utilities > Logs > Activity Log after the next cron run.

## All abbreviations

| Type + Tab | Generates |
|---|---|
| `whmodule` | provisioning module skeleton (MetaData, ConfigOptions, Create/Suspend/Unsuspend/Terminate) |
| `whaddon` | addon module skeleton (config, activate, deactivate, output) |
| `whhook` | hook: add_hook with closure |
| `whapi` | internal API call with result check |
| `whgateway` | payment gateway skeleton (MetaData, config, link) |
| `whregistrar` | domain registrar module skeleton (MetaData, config, register, nameservers) |
| `whcallback` | payment gateway callback file (validate invoice and transaction, log, add payment) |
| `whinvoicepaid` | InvoicePaid hook that loads the invoice through localAPI |
| `whcron` | DailyCronJob hook with a failure log so one error does not stop the cron |
| `whticket` | TicketOpen hook that loads the ticket through localAPI GetTicket with a failure log |
| `whclientadd` | ClientAdd hook that loads the client through localAPI GetClientsDetails with a failure log |
| `whorder` | AcceptOrder hook that loads the order through localAPI GetOrders with a failure log |
| `whinvoicecreated` | InvoiceCreated hook that loads the invoice through localAPI GetInvoice with a failure log |
| `whticketreply` | TicketAdminReply hook that loads the ticket through localAPI GetTicket with a failure log |
| `whticketuser` | TicketUserReply hook that loads the ticket through localAPI GetTicket with a failure log |
| `whclientarea` | ClientAreaPage hook that returns extra Smarty variables to the client area templates |
| `whcheckout` | ShoppingCartValidateCheckout hook that blocks the order by returning error messages |
| `whmodulecreate` | AfterModuleCreate hook that runs after a provisioning module creates a service |
| `whmodulesuspend` | AfterModuleSuspend hook that runs after a provisioning module suspends a service |
| `whmoduleterminate` | AfterModuleTerminate hook that runs after a provisioning module terminates a service |
| `whmoduleunsuspend` | AfterModuleUnsuspend hook that runs after a provisioning module unsuspends a service |
| `whmodulechangepackage` | AfterModuleChangePackage hook that runs after a provisioning module changes the package of a service |
| `whmodulechangepassword` | AfterModuleChangePassword hook that runs after a provisioning module changes the password of a service |
| `whdomainregister` | AfterRegistrarRegistration hook that runs after a registrar module registers a domain |
| `whdomainrenew` | AfterRegistrarRenewal hook that runs after a registrar module renews a domain |
| `whdomaintransfer` | AfterRegistrarTransfer hook that runs after a registrar module starts a domain transfer |
| `whinvoicecancelled` | InvoiceCancelled hook that loads the invoice through localAPI GetInvoice with a failure log |
| `whinvoicerefunded` | InvoiceRefunded hook that loads the invoice through localAPI GetInvoice with a failure log |
| `whticketclose` | TicketClose hook that loads the ticket through localAPI GetTicket with a failure log |
| `whclientedit` | ClientEdit hook that loads the client through localAPI GetClientsDetails with a failure log |
| `whinvoicereminder` | InvoicePaymentReminder hook that loads the invoice through localAPI GetInvoice with a failure log |
| `whinvoiceunpaid` | InvoiceUnpaid hook that loads the invoice through localAPI GetInvoice with a failure log |
| `whclientclose` | ClientClose hook that loads the client through localAPI GetClientsDetails with a failure log |
| `whordercancel` | CancelOrder hook that loads the order through localAPI GetOrders with a failure log |
| `whaddonactivation` | AddonActivation hook that loads the service addons through localAPI GetClientsAddons with a failure log |
| `whticketstatus` | TicketStatusChange hook that loads the ticket through localAPI GetTicket with a failure log |
| `whclientpassword` | ClientChangePassword hook that loads the client through localAPI GetClientsDetails with a failure log |
| `whorderpaid` | OrderPaid hook that loads the order through localAPI GetOrders with a failure log |
| `whorderfraud` | FraudOrder hook that loads the order through localAPI GetOrders with a failure log |
| `whorderaccept` | AcceptOrder hook that loads the order through localAPI GetOrders with a failure log |
| `whorderpending` | PendingOrder hook that loads the order through localAPI GetOrders with a failure log |
| `whserviceedit` | ServiceEdit hook that loads the service through localAPI GetClientsProducts with a failure log |
| `whdomainedit` | DomainEdit hook that loads the domain through localAPI GetClientsDomains with a failure log |
| `whticketnote` | TicketAddNote hook that loads the ticket through localAPI GetTicket with a failure log |
| `whticketpriority` | TicketPriorityChange hook that loads the ticket through localAPI GetTicket with a failure log |
| `whticketdept` | TicketDepartmentChange hook that loads the ticket through localAPI GetTicket with a failure log |
| `whticketsubject` | TicketSubjectChange hook that loads the ticket through localAPI GetTicket with a failure log |
| `whticketflag` | TicketFlagged hook that loads the ticket through localAPI GetTicket with a failure log |
| `whticketopenadmin` | TicketOpenAdmin hook that loads the ticket through localAPI GetTicket with a failure log |
| `whinvoicegateway` | InvoiceChangeGateway hook that loads the invoice through localAPI GetInvoice with a failure log |
| `whinvoicepayment` | AddInvoicePayment hook that loads the invoice through localAPI GetInvoice with a failure log |
| `whinvoicelatefee` | AddInvoiceLateFee hook that loads the invoice through localAPI GetInvoice with a failure log |
| `whinvoicetotal` | UpdateInvoiceTotal hook that loads the invoice through localAPI GetInvoice with a failure log |
| `whtransaction` | AddTransaction hook that loads the transaction through localAPI GetTransactions with a failure log |
| `whserviceaddon` | AddonActivated hook that loads the service addons through localAPI GetClientsAddons with a failure log |
| `whquoteaccept` | AcceptQuote hook that loads the quote through localAPI GetQuotes with a failure log |
