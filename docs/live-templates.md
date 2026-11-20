# Live templates

WHMCS Toolkit ships 56 live templates. Open a `.php` file, type the abbreviation, press Tab. Nothing to configure. In PhpStorm: Settings > Editor > Live Templates > WHMCS lists them all and lets you edit the body.

## Example: do something when an invoice is paid

1. Create `includes/hooks/paid.php` in your WHMCS folder and start it with `<?php` on the first line, then press Enter. The templates only expand inside PHP code, so in a completely empty file nothing happens (see Troubleshooting below).
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

## Example: flag an urgent ticket for staff when it is opened

1. Create `includes/hooks/tickets.php`, type `whticket` and press Tab. You get a `TicketOpen` hook that already loads the ticket with `localAPI('GetTicket', ...)` inside a `try`, with the cursor on the line after it.
2. `$vars` carries what WHMCS passes to `TicketOpen`: `ticketid`, `userid`, `deptid`, `deptname`, `subject`, `message` and `priority`. Use them directly, so you do not need the `GetTicket` result for a simple rule. Replace the cursor line with:

```php
if ($vars['priority'] === 'High') {
    localAPI('AddTicketNote', [
        'ticketid' => $vars['ticketid'],
        'message' => 'Opened as High priority in ' . $vars['deptname'] . ': check first.',
    ]);
}
```

3. Open a ticket as a client with priority High. The staff-only note appears in the ticket under Support > Tickets. If the call fails, the `catch` block writes `TicketOpen hook failed: ...` to Utilities > Logs > Activity Log and the ticket is still created.
4. For the other ticket events use `whticketreply` (admin reply), `whticketuser` (client reply), `whticketclose`, `whticketstatus`, `whticketpriority` and `whticketdept`; they generate the same shape. See the table below.

## Example: leave a note on the client when an order is accepted

1. Create `includes/hooks/orders.php`, type `whorder` and press Tab. You get an `AcceptOrder` hook that loads the order with `localAPI('GetOrders', ...)` inside a `try`, with the cursor on the line after it. WHMCS passes only `orderid` to this hook, so the template fetches the rest for you.
2. `GetOrders` answers with a list: the order you asked for is `$order['orders']['order'][0]`, with `userid`, `amount`, `paymentstatus` and `status`. Replace the cursor line with:

```php
$row = $order['orders']['order'][0] ?? null;
if ($row && $row['paymentstatus'] !== 'Paid') {
    localAPI('AddClientNote', [
        'userid' => $row['userid'],
        'notes' => 'Order #' . $vars['orderid'] . ' accepted while still unpaid (' . $row['amount'] . ').',
    ]);
}
```

3. Accept an unpaid order under Orders > Pending Orders. The note shows up on the client's Notes tab. If the lookup or the note fails, the `catch` block writes `AcceptOrder hook failed: ...` to Utilities > Logs > Activity Log and the acceptance still goes through.
4. For the other order events use `whorderpaid`, `whordercancel`, `whorderfraud`, `whorderpending` and `whorderaccept`; they generate the same shape. See the table below.

## Example: tag new clients that need a tax ID

1. Create `includes/hooks/clients.php`, type `whclientadd` and press Tab. You get a `ClientAdd` hook that loads the client with `localAPI('GetClientsDetails', ...)` inside a `try`, with the cursor on the line after it. WHMCS already passes `country`, `companyname`, `email` and `client_id` in `$vars`, so you can decide without the lookup.
2. Replace the cursor line with:

```php
if ($vars['country'] === 'BR' && $vars['companyname'] === '') {
    localAPI('AddClientNote', [
        'userid' => $vars['client_id'],
        'notes' => 'Brazilian individual: ask for CPF before the first invoice.',
    ]);
}
```

3. Register a test client with country Brazil and no company name. The note shows up on the client's Notes tab. If the lookup or the note fails, the `catch` block writes `ClientAdd hook failed: ...` to Utilities > Logs > Activity Log and the signup still completes.
4. Misspell a key, for example `$vars['conpanyname']`, and the WHMCS inspection flags it with a quick fix to `companyname`. For other client events use `whclientedit`, `whclientclose` and `whclientpassword`; they generate the same shape. See the table below.

## Example: log every domain your registrar module registers

1. Create `includes/hooks/domains.php`, type `whdomainregister` and press Tab. You get an `AfterRegistrarRegistration` hook that already reads `$domainId = (int) $vars['params']['domainid'];`, with the cursor on the next line. `$vars['params']` is the same array your registrar module receives, so `sld`, `tld` and `regperiod` are there too.
2. Replace the cursor line with:

```php
logActivity(sprintf(
    'Domain %s.%s registered for %d year(s) (domain #%d)',
    $vars['params']['sld'],
    $vars['params']['tld'],
    $vars['params']['regperiod'],
    $domainId
));
```

3. Register a domain through your registrar module (for example from Clients > Domain Registration > Register). The line shows up under Utilities > Logs > Activity Log. The hook runs only after the registrar reported success; for failures WHMCS fires `AfterRegistrarRegistrationFailed`, which has no template here.
4. For the other registrar events use `whdomainrenew` and `whdomaintransfer`; they generate the same shape. See the table below.

## Example: record an accepted quote on the client

1. Create `includes/hooks/quotes.php`, type `whquoteaccept` and press Tab. You get an `AcceptQuote` hook that loads the quote with `localAPI('GetQuotes', ...)` inside a `try`, with the cursor on the line after it. WHMCS passes `quoteid` and the new `invoiceid` in `$vars`.
2. `GetQuotes` answers with a list: the quote you asked for is `$quotes['quotes']['quote'][0]`, with `userid`, `subject` and `total`. Replace the cursor line with:

```php
$row = $quotes['quotes']['quote'][0] ?? null;
if ($row) {
    localAPI('AddClientNote', [
        'userid' => $row['userid'],
        'notes' => 'Quote #' . $vars['quoteid'] . ' "' . $row['subject'] . '" accepted, invoice #' . $vars['invoiceid'] . '.',
    ]);
}
```

3. Accept a quote from the client area. The note shows up on the client's Notes tab. If the lookup or the note fails, the `catch` block writes `AcceptQuote hook failed: ...` to Utilities > Logs > Activity Log and the acceptance still goes through. The response shape comes from the WHMCS API reference; check it against your version with `logActivity(json_encode($quotes))` the first time.

## Example: flag large invoices for review when they are created

1. Create `includes/hooks/invoices.php`, add `<?php` on the first line, type `whinvoicecreated` and press Tab. You get an `InvoiceCreated` hook that loads the invoice with `localAPI('GetInvoice', ...)` inside a `try`, with the cursor on the line after it. WHMCS passes `invoiceid` in `$vars`.
2. `GetInvoice` answers with `userid`, `total`, `duedate` and `status` at the top level. Replace the cursor line with:

```php
if ((float) $invoice['total'] >= 1000) {
    logActivity('Large invoice #' . $vars['invoiceid'] . ' (' . $invoice['total'] . ', due ' . $invoice['duedate'] . ') created for client #' . $invoice['userid'] . '.');
}
```

3. Generate an invoice of 1000 or more (Billing > Create Invoice, then publish it so it leaves Draft). The line shows up in Utilities > Logs > Activity Log. Change `1000` to your own threshold, or swap `logActivity` for `localAPI('AddClientNote', ...)` to keep the flag on the client. If the lookup fails, the `catch` block writes `InvoiceCreated hook failed: ...` to the same log and the invoice is created anyway. Field names come from the WHMCS API reference; confirm them on your version with `logActivity(json_encode($invoice))` the first time.

## Example: record why a service was suspended

1. Create `includes/hooks/provisioning.php`, add `<?php` on the first line, type `whmodulesuspend` and press Tab. You get an `AfterModuleSuspend` hook with `$serviceId` and `$reason` already read from `$vars['params']` (`serviceid` and `suspendreason`), and the cursor on the line after them. WHMCS runs this hook only after the provisioning module's suspend function succeeded.
2. Replace the cursor line with:

```php
logActivity('Service #' . $serviceId . ' (' . ($vars['params']['domain'] ?? 'no domain') . ') suspended: ' . ($reason !== '' ? $reason : 'no reason given') . '.');
```

3. Suspend a service (Clients > Products/Services > Module Commands > Suspend, with a reason typed in the box). The line shows up in Utilities > Logs > Activity Log. `whmodulecreate`, `whmoduleunsuspend` and `whmoduleterminate` give the same hook for the other module commands, so you can log each step of a service's life the same way. `$vars['params']` is the same array your module functions receive, so `username`, `serverip` and your own `configoption1` are there too; Ctrl+Space inside `$vars['params']['...']` lists them. The suspend reason is only filled when the command comes from the admin area or the automation with a reason; the `?? ''` in the template covers the cases where WHMCS sends none.

## Example: leave a note on the client when an invoice is refunded

1. Create `includes/hooks/invoices.php` (or reuse it), make sure `<?php` is on the first line, type `whinvoicerefunded` and press Tab. You get an `InvoiceRefunded` hook that loads the invoice with `localAPI('GetInvoice', ...)` inside a `try`, with the cursor on the line after it. WHMCS passes `invoiceid` in `$vars`.
2. `GetInvoice` answers with `userid`, `total` and `status` at the top level. Replace the cursor line with:

```php
localAPI('AddClientNote', [
    'userid' => $invoice['userid'],
    'notes' => 'Invoice #' . $vars['invoiceid'] . ' refunded (' . $invoice['total'] . '). Check the client before reactivating services.',
]);
```

3. Refund a paid invoice (Billing > Transactions or the invoice's Refund button). The note shows up on the client's Notes tab. If the lookup or the note fails, the `catch` block writes `InvoiceRefunded hook failed: ...` to Utilities > Logs > Activity Log and the refund still goes through. Field names come from the WHMCS API reference; confirm them on your version with `logActivity(json_encode($invoice))` the first time. `whinvoicecancelled`, `whinvoicereminder` and `whinvoiceunpaid` generate the same shape for the other invoice events.

## Troubleshooting: Tab does nothing

- The cursor must be in PHP code: after `<?php`, not in an empty file, an HTML block, a comment or a string. Add `<?php` on the first line and try again.
- The abbreviation must be the last word before the cursor, typed exactly (`whinvoicepaid`, not `wh invoicepaid`). If the completion popup is open, press Tab again or Esc first.
- Check Settings > Editor > Live Templates > WHMCS: the group must be ticked, and each template must list PHP under its applicable contexts.
- Completion and typo inspection (Ctrl+Space, Alt+Enter) need the PHP plugin, which PhpStorm bundles and IntelliJ IDEA Ultimate needs installed from Settings > Plugins.

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
