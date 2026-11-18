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
