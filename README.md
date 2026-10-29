# Hosting Your First Static Website in Azure

A one-page website served straight from an Azure storage account, with no web server to build or patch. I built it in the Azure portal, checked it from the browser and from PowerShell, looked at what the storage account refuses to show anonymously, and then deleted everything and confirmed it was gone.

You do not need any Azure experience to follow this. Each step says what it does, what to do, and what you should see when it works.

## What this lab is for

A static website is a set of files that never change when someone visits: HTML, images, style sheets. Nothing has to run code to answer a request. Azure Storage can serve files like that directly, so the whole "server" is one storage account.

- **Host a page:** upload one `index.html` file and open it from a public address.
- **See what an empty site looks like:** before the upload, the address shows an Azure error page, not a blank one.
- **Learn the two addresses:** a storage account has a website address and a blob address, and they follow different access rules.
- **Take it down and prove it:** delete the resource group, then check from Azure and from the web that nothing is left.

## Words you will see

| Word | What it means here |
|---|---|
| Resource group | A folder in Azure that holds related resources. Deleting the folder deletes everything inside it. |
| Storage account | An Azure resource that stores files (blobs), tables and queues. This lab uses only blobs. |
| Blob | A file stored in a storage account. |
| Container | A folder inside a storage account that holds blobs. |
| `$web` | The container Azure creates for you when you turn on static website hosting. The website serves files from here. |
| Static website endpoint | The public address of the website. It looks like `https://<account>.z19.web.core.windows.net/`. |
| Blob endpoint | A second address of the same storage account, `https://<account>.blob.core.windows.net/`, used to reach blobs directly. |
| Index document | The file the site shows when someone opens the bare address. Here, `index.html`. |
| Error document | The file the site shows for a missing page. Here, `404.html`. |
| DNS (Domain Name System) | The lookup that turns a name such as `stlab01[your name].z19.web.core.windows.net` into an address a computer can connect to. |

## Architecture

```
                       static website endpoint
  Browser  ----------> https://stlab01[your name].z19.web.core.windows.net/
                                    |
                                    v
        Resource group: rg-lab01-[your name}   (Central US)
        +-------------------------------------------------+
        |  Storage account: stlab01[your name}              |
        |  StorageV2, Standard_LRS                        |
        |                                                 |
        |    $web container                               |
        |      index.html                                 |
        +-------------------------------------------------+
                                    ^
  Browser  ----------> https://stlab01[your name].blob.core.windows.net/$web/index.html
                       blob endpoint: refused (PublicAccessNotPermitted)
```

## What's inside

```
azure-static-website-lab/
  README.md          this file
  site/
    index.html       the page that gets uploaded to $web
```

## Before you start

- An Azure subscription and permission to create resources in it.
- The Azure CLI installed, to run the checks.
- PowerShell and a text editor such as VS Code.

Sign in first, and do it again in any new terminal:

```powershell
az login
```

## Step 1: Create the resource group

**What this step does.** It makes the folder that holds everything in the lab. One folder means one delete at the end.

**What to do.** In the portal, open **Resource groups**, click **Create**, and use:

- Resource group name: `rg-lab01-[your name}  `
- Region: Central US

Click **Review + create**, then **Create**. The region of a resource group is where Azure keeps the group's own information. The resources inside it can sit in other regions, so this choice does not force the storage account's region.

**What you should see.** Check it from the terminal:

```powershell
az group show -n rg-lab01-[your name}   -o table
```

It prints the location `centralus` and the name `rg-lab01-[your name}`. I typed the name in lowercase and Azure still found the group. It kept the capital D I used when I created it.

## Step 2: Create the storage account

**What this step does.** It creates the account that will hold and serve the site.

**Before you run it.** A storage account name becomes part of a public address. Do you expect the name to have to be unique only inside your subscription, or across all of Azure?

**What to do.** In the portal, open **Storage accounts**, click **Create**, and use:

- Resource group: `rg-lab01-[your name}  `
- Storage account name: `stlab01[your name}  `
- Region: Central US

The name must be globally unique, because it is part of the website's public DNS name. Pick your own name. Click **Review + create**, then **Create**.

**What you should see.** Check the real settings from Azure and not from the form:

```powershell
az storage account list -g rg-lab01-[your name}   --query "[].[name,location,kind,sku.name,allowBlobPublicAccess]" -o tsv
```

It printed `stlab01[your name}  centralus StorageV2 Standard_LRS False`. That is the account name, region, kind, redundancy type and whether anonymous blob access is allowed at the account level. It was `False`.

## Step 3: Look before you turn the feature on

**What this step does.** It records the starting state, so the change in the next step is visible.

**Before you run it.** When you turn on static website hosting, will Azure create `index.html` and `404.html` for you?

**What to do.** Open the storage account. Under **Data storage**, open **Containers**. Then open **Static website**.

**What you should see.** **Containers** lists only `$logs`. **Static website** says **Disabled**.

I predicted that Azure would create the two HTML files. That was wrong. See the next step.

## Step 4: Turn on static website hosting

**What this step does.** It makes Azure serve a container as a website and tells it which files to use.

**What to do.** On the **Static website** page, select **Enabled** and set:

- Index document name: `index.html`
- Error document path: `404.html`

Click **Save**. The page then shows a **Primary endpoint**. That is the address of the site. Settings can take up to 30 seconds to apply.

File names in `$web` are case-sensitive. The first time through I typed `Index.html` with a capital I. I noticed it when I was asked to read the field back, and I changed it to `index.html`. A capital I would have made Azure look for a file that does not exist.

**What you should see.** **Containers** now lists `$logs` and `$web`. Azure created the empty `$web` container. It did not create any files in it.

## Step 5: Open the site while it is empty

**What this step does.** It shows what a visitor sees when the feature is on but there is nothing to serve.

**Before you run it.** `$web` is empty. What will the browser show: a blank page, or something else?

**What to do.** Open the Primary endpoint in a browser.

**What you should see.** I predicted a blank page. It was an error page from Azure, a 404 page, not a blank one. The container was empty and `404.html` did not exist either, so I cannot say which of the two produced it.

## Step 6: Create the page and upload it

**What this step does.** It puts the content in `$web`.

**What to do.** Create `site/index.html` with this content. The file name is lowercase and the file is plain text.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>My First Cloud Site</title>
  <style>
    body { font-family: sans-serif; text-align: center; margin-top: 50px; background-color: #f0f0f0; }
    h1 { color: #0078d4; }
  </style>
</head>
<body>
  <h1>Hello from the Cloud!</h1>
  <p>This site is hosted on Azure Blob Storage.</p>
  <p>Deployed by: Demarcus</p>
</body>
</html>
```

In the portal, open the `$web` container and upload `index.html`.

**What you should see.** The blob list for `$web` shows `index.html`.

## Step 7: Check that the site works

**What this step does.** It checks the result two ways, from a browser and from the terminal.

**What to do.** Open the Primary endpoint in a browser. Then run this, with your own account name:

```powershell
(Invoke-WebRequest -Uri https://stlab01[your name].z19.web.core.windows.net/ -UseBasicParsing).StatusCode

```

**What you should see.** The browser shows "Hello from the Cloud!" and the command prints `200`. `200` is the code for "the request worked". A page drawn in a browser tells you less than a status code from Azure does.

## Step 8: Try the other address

**What this step does.** It shows that the same files behave differently through the second address.

**What to do.** Open this address in a browser:

```
https://stlab01[your name].blob.core.windows.net/$web/index.html
```

**What you should see.** An XML error with the code `PublicAccessNotPermitted`. The file is there, and the website address serves it, but the blob address refuses anonymous visitors.

Microsoft's static website page describes the same split: files in `$web` are served anonymously through the website address, and turning off anonymous access on the storage account does not affect that. The blob address follows the container's own access level. That matches what I saw on this account, where the account-level setting was `False` and the site still loaded. [Microsoft's documentation](https://docs.microsoft.com/azure/storage/blobs/storage-blob-static-website) has the details.

Chrome also showed a "Dangerous site" warning on this blob address. I do not know why. I clicked through it to read the XML error, entered nothing and downloaded nothing. I would not recommend doing that. If a browser warns you about an address, stop and look at the address before you decide.

## Step 9: Tear it down

**What this step does.** It removes everything the lab created. Deleting the resource group deletes the storage account and the file inside it. There is no undo.

**Before you run it.** After the group is gone, what will the website address do?

**What to do.** In the portal, open **Resource groups**, open `rg-lab01-[your name]`, click **Delete resource group**, type the name to confirm, and click **Delete**. Wait for the notification. Then check from the terminal:

```powershell
az login
az group exists -n rg-lab01-[your name]
```

Then run the website check again:

```powershell
(Invoke-WebRequest -Uri https://stlab01[your name].z19.web.core.windows.net/ -UseBasicParsing).StatusCode
```

**What you should see.** `az group exists` printed `false`. The website check did not print a number. It printed an error: "The remote name could not be resolved". No server answered at all. That is different from the 404 page in Step 5, where the storage account existed and said the file was missing.

I predicted a page saying it does not exist. The real result was a failed name lookup. I did not look at the DNS records, so I am describing what PowerShell reported and not why.

## Why I built it this way

- **One resource group:** one delete removes everything, and one `az group exists` proves it.
- **Verify with Azure, not with the form:** the region, kind and access setting came from `az`, and the page was checked with a status code as well as a browser.
- **Predict first:** I wrote down what I expected before each step. Three of my predictions were wrong, and those are the parts I remember best.
- **No secrets and no IDs:** nothing here needs a password or key, and no tenant or subscription details appear anywhere in this repository.

## What I took from this

My three wrong predictions each taught the same thing: Azure's default behavior is smaller than I assumed. Turning on the feature creates an empty container, not a starter site. An empty site shows an Azure error page, not a blank one. A deleted account does not return a "gone" page, because there is nothing left to answer.

The result I would have missed without checking is the second address. I had been thinking of the account as having one door. It has two, and they follow different rules. The account-level setting `allowBlobPublicAccess = False` did not stop the website, and the documentation says it should not. Anyone who treats that setting as "nothing in this account is public" is wrong for `$web`.

