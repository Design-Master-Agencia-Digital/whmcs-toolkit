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
