# Change log

## 0.6.1

This is an architecture rewrite that removes the manually written file I/O functions in favor of using the CSV crate.

## 0.6.0

*This was originally 1.0.0-alpha.1 but was retroactively made 0.6.0 as part of the architecture rewrite.*

### Personal Subcommand

```text
USAGE:
    cra pers [OPTIONS] --name <name> --description <description>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -d, --description <description>    
    -n, --name <name>                  
    -r, --reference <reference>
```

Personal, or ``pers``, is a simpler variant of the order subcommand. This subcommand is meant for record keeping of personal projects and lacks any economy features. This feature replaces Requests subcommand but can be used for the same purpose. It expects a name, description and an optional reference. The CSV header will include a column for references even when is none supplied.

The code block that handled appending and reading files from ``csv_manager()`` was moved to its own function, called ``content_manager()``, in order to this subcommand possible. ``order_manager()`` as well as ``personal_manager()``, for this new subcommand, both use the new function.

## 0.5.2

- Wrapped more values in quotations for better flexibility.

## 0.5.1

- Fees are in quotations now because I forgot some currencies use commons instead of dots for their decimal points, e.g. [Swedish crown](https://en.wikipedia.org/wiki/Swedish_krona). See [usage](Usage.md) for examples.
- Renamed ``/artm`` directory to ``/CommissionRecords``

## 0.5.0

- Removed passive tones from README

### Breaking changes

- ``--client`` option has been changed to ``--buyer`` to make room for the ``--currency`` option.
- Renamed from Art Manager to Commission Records for Artists, or "CRA" for short.

### Note

While an embedded database had been planned for CRA (and still on the drawing board), CSVs have been proven to be sufficient for data exchange and storage. It can be opened in any spreadsheet software that supports format for further editing in data exchange or imported into a database for storage. Much better than reinventing the wheel with JSON.

As this breaking change is more of a refinement and improvement built upon previous version's changes, 0.5 will likely be the last in 0.x series - finally.

## 0.4.0

### Breaking changes

Art Manager has been completely rewritten to use Clap's derive macros. This dramatically reduces the complexity with the lines of code have been reduced from 499 to 116.

However, this change also brings back subcommands. Similar to 0.3, the ``order`` subcommand combines YCHs and commissions. ``-y`` or ``--ych`` is an option - not a flag - that requires the name of the YCH. ``--slot`` is still required.

Finally, the custom JSON-based format has been dropped in favor of CSV, for use in office applications. Because CSV files are more compact, each order now has their own respective file. I.e. YCH has ``ych.csv``, commissions have ``commission.csv``, and so on.

### Other changes

- ``--contact`` has been removed because it conflicts with ``--client``
    - This may return under another name
- ``--name`` was removed because ``--description`` is more important 
- ``--price`` has been renamed ``--fee`` due to conflicts with ``--payment``
- Fixed licensing confusion
- Request has been removed because those are personal

## 0.3.0

- ``.art[ycr]`` replaces the former ``.artm[ycr]`` format.
- Submissions and file creations are now logged.
- YCH, Commission, and Request subcommands, flags or values are no longer required! Finally. The program assumes what you want based on the options selected - as it should be, imo. For example, if ``--payment`` and ``--slot`` isn't used then it assumes you're referring to a Request. Raffle remains it's own subcommand with different arguments.
- ``--art`` was renamed to ``--name`` and has been made global for use with the ``raffe`` subcommand. However, ``--cust`` renames the same.
- Secure ID and Raffle methods now generate their own respective IDs based on Sha256.


### .artm* is now .art*

The current ``.artm[ycr]`` formats has been redone as ``.art[ycr]``. The older format were based on three independent implementations that all did very similar things but lacked room for change. This worked as a proof-as-concept but left no room to grow. This new format merges all three by making every field optional and won't show up, if left empty, with the exception of name, version, date and Id.

```json
{
  "id": "",
  "version": "",
  "name": "",
  "price": "",
  "ticket": "",
  "slot": "",
  "description": "",
  "reference": "",
  "customer": {
    "name": "",
    "contact": "",
    "payment": ""
  }
}
```
And so the previous YCH example found in the usage page becomes...
```json
{
  "id": "c152c08a-1250-45c1-a51d-a38354804933",
  "date": "2019-04-05T13:53:43.670815600-04:00",
  "version": "0.1",
  "name": "Synthesize",
  "category": "YCH",
  "slot": "4",
  "price": "$25",
  "reference": "https://www.furaffinity.net/view/20700210/",
  "customer": {
    "name": "Bessie Hettinger",
    "contact": "Jack.Torphy75",
    "payment": "31VLNZXfcpoA68wPRuWSdrmT3jv5k"
  }
}
```
We're able to create as many variations of this format as we want using the [builder pattern](https://en.wikipedia.org/wiki/Builder_pattern), this method is common in a lot of Rust libraries and frameworks. This takes care of a lot of the heavy lifting and allows for any number of combination to exist.

```rust
// Example
    if let Err(err) = Art::new()
        .price(price)
        .name(name)
        .category(Category::YCH)
        .slot(slot)
        .reference(reference)
        .customer(Customer::new()
            .name(cust_name)
            .payment(pay)
            .contact(cont))
        .secure_id()
        .debug(debug)
        .write_file() {
        println!("{}: {}", ERROR_MSG, err);
        process::exit(EXIT_CODE);
    }
```

### Secure Ids

Up until now, Art Manager's Ids have been random UUIDs. 0.3 introduces secure Ids that use Sha256 for use as a checksum. These come in two variations, both based on *local time*: normal and raffle.

- Normal: ``artm:[year][month][day][hour][minute][name]``
- Raffle: ``artm:[year][month][day][hour][minute][name][ticket][slot]``

## 0.2.10

- Request & Commissions are now flags since they share a lot of the same data. YCH is now a subcommand that now carries the Raffle subcommand. Due to this new arrangement, all current optional arguments are now global including the one in the YCH subcommand, so it can be accessed by the Raffle.

## 0.2.5

- Request feature now be accessed.
- Request, Commission and YCH flags now conflict with each other.
- Argument changes: "order" is now "art", as in the art's name, "pay" is now payment", and "cost" is now "price."
- New arguments include "reference" and "description", these names were shortned to "ref" and "desc", respectfully. Reference is used for YCHs while description is used for Commissions and Requests.
- Added the raffle subcommand. It'll crash if you use all but --help.

## 0.2.0

- ``--debug`` flag prints the output to the screen as oppose to a file.
- Reverted back to using subcommands again now that everything was moved to config file.
- Client is now customer.
- Username is now contact.
- Customer, contact, payment and tickets options are now abbreviated.
- Debug flag is now global.