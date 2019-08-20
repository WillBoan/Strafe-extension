<!-- TOC -->

- [Overview](#overview)
  - [What Strafe does](#what-strafe-does)
  - [From a dev standpoint — Things that should be maintained](#from-a-dev-standpoint--things-that-should-be-maintained)
    - [Priority things](#priority-things)
    - [Other things that may need some maintenance](#other-things-that-may-need-some-maintenance)
- [Using Strafe](#using-strafe)
  - [Getting started](#getting-started)
    - [Installing](#installing)
    - [Opening](#opening)
  - [Why Strafe matters](#why-strafe-matters)
  - [Strafe's intended use](#strafes-intended-use)
- [More detailed info about Strafe](#more-detailed-info-about-strafe)
  - [File structure](#file-structure)
  - [Dependencies](#dependencies)
  - [Architecture](#architecture)
    - [Caselists](#caselists)
    - [Projects](#projects)
  - [UI](#ui)
    - [Tabs and data tables](#tabs-and-data-tables)
    - [The searches table](#the-searches-table)
    - [The bottom bar buttons](#the-bottom-bar-buttons)
    - [Modals](#modals)
  - [Functionalities](#functionalities)
    - [Corrections](#corrections)
      - [Basic formatting](#basic-formatting)
      - [Party replacements](#party-replacements)
      - [Title casing](#title-casing)
    - [Searches](#searches)

<!-- /TOC -->

# Overview

Strafe is a Google Chrome extension, designed to facilitate some of the legal research team's activities.

### What Strafe does

Strafe operates on caselists. Here’s what Strafe can do with a caselist:

* Correct case name formatting
* Find duplicate cases
* Rank cases in order of codeability

### From a dev standpoint — Things that should be maintained

##### Priority things

* The `titleCaseWord` function in nlp.js — The rules will need maintenance
* Handling of special characters should be maintained. This may involve adding new special character regex/replacements to the `special_character_mappings` array in nlp.js. Ultimately, it might be better to just revamp the special character handling.

##### Other things that may need some maintenance

In the nlp.js module:
* `parseCaseName` function
  - Initial flagging of case names — New conditions may need to be added
  - `court_mappings` and `court_mappings_before_comma` {arrays} — New court mappings can be added
* `checkIfDuplicates` function — The rules may need tweaking
* `cleanCaseString` function — New formatting rules may need to be added; the regex may need to be tweaked a bit
* `correctPartyName` function
  - `party_replacements` {array} — New regex/replacement pairs can be added
* `titleCase` function
  - `title_case_exceptions` {array} — More joiner words (eg 'a', 'the') may need to be added
  - `acronyms` {array} — More instances may need to be added of words that should be kept upper-case (eg 'USA', 'NASA')

---

# Using Strafe

### Getting started

##### Installing

Installing and running Strafe requires Google Chrome.

Steps:
1. Download from Github and unzip.
2. Go to [your Chrome extensions page](chrome://extensions/). In the top right, turn on developer mode. In the top left, click 'Load unpacked', then select the extension folder you just unzipped (it will be named 'strafe-master').

All done! The Strafe icon will now be in the top right of Chrome.

##### Opening

To open Strafe, click on the Strafe icon in the top right of Chrome, then click ‘Open’. Strafe can also be disabled from this menu (if disabled, it won’t try to run on cases you open).

![Clicking the Strafe icon](images/readme-images/strafe-icon-and-popup.png)

### Why Strafe matters

When the legal research team creates and codes a classifier, there are various problems that arise:

* **Errors in case names**: Case names are often intended to be used as unique identifiers for cases. So if a case name has an error, users might not be able to find the right case. It also looks sloppy.
* **Duplicates**: When a caselist is created, it usually has some cases that are actually duplicates of each other. If we don't notice this, we'll double-code them. This costs time, skews our data, and makes our product look sloppy and confusing.
* **Culling**: The culling process (the process of manually removing non-relevant cases from a caselist) takes a significant amount of time.

Strafe addresses all three of these problems. Indeed, here’s what Strafe can do with a caselist:

* Correct case name formatting
* Find duplicate cases
* Rank cases in order of codeability

### Strafe's intended use

Strafe has two intended use-cases.

**Caselist Creation**: An associate using Strafe to facilitate the process of constructing a caselist for coding and uploading it to Wrike.

**Quality Assurance**: Any legal researcher using the extension as a data clean-up tool, using it on any caselist to check for errors, duplicates, etc.

|                                |   **Caselist Creation**    |  **Quality Assurance**  |
|--------------------------------|:--------------------------:|:-----------------------:|
| **Who uses it**                |         Associates         | Analysts and associates |
| **When is it used**            |       Before coding        |      After coding       |
| **Multiple rulings and higher/lower decisions in case names** |             No             |          Maybe          |
| **Caselist import format**<sup id="a1">[1](#f1)</sup> | Database copy-paste format |   Blue J data export    |

<sup id="f1">[1](#a1)</sup> You can find examples of the caselist import formats in the 'example-spreadsheets' folder.

---

# More detailed info about Strafe

### File structure

- **manifest.json** — An info file that is required for any Chrome extension, dictating when and which files the extension should run.
- **options-page** — Contains the code for the options page, which is the main page where Strafe is used.
- **case-page** — Contains the code that runs when the user opens a case. Contains the code that actually runs searches (if the user is using Strafe's search functionality)
- **images**
- **libraries** — Contains Strafe's dependencies ([see below](#dependencies))
- **modules**
  - nlp.js — Contains all of the tools for parsing and correcting case names and identifying duplicates.
  - storage.js — Contains the functions relating to the data stored in Chrome local storage.
  - utilities.js — Contains general utility functions, such as quicksort and math-related functions.

### Dependencies

* **jQuery** is used throughout, mostly for interacting with the HTML DOM.
* [**PapaParse**](https://www.papaparse.com/) is used for reading CSVs.
* [**SheetJS xlsx**](https://docs.sheetjs.com/#sheetjs-js-xlsx) is used for reading and writing excel files, and generally handling spreadsheets.

### Architecture

##### Caselists
Strafe operates on caselists. A caselist has one or more cases on it. A caselist could be a list of just case names; or it could also have other data (url, coder, actual codeability, etc.).

Users import the caselist they want to use Strafe on.

##### Projects
Strafe allows the user to create different projects. Each project has its own data—a caselist, and possibly searches and results.

The _active project_ is the project that is currently being displayed, and whose searches are being run on cases that are opened.

### UI

##### Tabs and data tables

The main portion of the window displays a number of tabs, each with a table. These tables display various data for the active project.

Two tabs are always there:
* **All Data** — Shows you the data from the entire caselist you have imported. Note that flagged cases won’t show up in this table, and only one of a set of duplicates will show up.
* **Flagged** — Shows you the data points that Strafe couldn’t recognize as case names. This is usually due to significant irregularities in formatting or content.

Other tabs are sometimes available, depending on the active project's settings:
* **Corrections** — Shows you formatting errors Strafe has found in the imported caselist, along with suggested corrections and reasons for the corrections. ([See below for how the corrections functionality works.](#corrections))
* **Duplicates** — Shows you cases that Strafe has determined are likely to be duplicates. If more than one duplicate is found, duplicates beyond the first are placed directly under the first in the table. ([See below for how the duplicates functionality works.](#duplicates))

##### The searches table

The searches table appears if the searches setting is turned on for the active project. This table allows the user to write, edit, and change the weights of the weighted regex searches ([see below for more detail about the searches functionality](#searches)).

##### The bottom bar buttons

* Import — Imports a caselist from a spreadsheet of the user's choice
* Export [tab name] — Exports the displayed tab as a .xlsx file
* Export (Wrike) — Exports the caselist in the format for import to Wrike
* Projects — Brings up the projects modal
* Settings — Brings up the settings modal

##### Modals

* The projects modal — allows the user to navigate, view, create, edit, and delete projects
* The settings modal — allows the user to change the non-project-specific settings

### Functionalities

#### Corrections

The corrections functionality fixes case names. Strafe makes three main types of corrections:
* Basic formatting (spacing, punctuation, etc)
* Party replacements
* Title casing

###### Basic formatting

```
Original:  Doe v. Wilson , 26 TC Memo 706 (T.C., 2007)
Corrected: Doe v. Wilson, 26 TC Memo 706 (T.C., 2007)
```

###### Party replacements

Certain government parties appear in many cases. For example, the Commissioner of Internal Revenue is the second party in most US tax cases. However, it is written in various forms, such as Commissioner, Comm'r, Com'r, CIR, C.I.R., etc.

It is the legal research team's policy to make this consistent. For the Commissioner of Internal Revenue, the unified form is "Commissioner".

Strafe is able to perform such party replacements on case names.

```
Original:  Smith v. CIR, 13 TC Memo 1002 (T.C., 2011)
Corrected: Smith v. Commissioner, 13 TC Memo 1002 (T.C., 2011)
```

###### Title casing

Case names sometimes have party names in all caps. This is strange and unnecessary, and it looks sloppy, so we should fix it.

Strafe converts every case name to **title case**—the casing that is normally used for titles. For example, `The Lord of the Rings` is in title case.

Conversion to title case is non-trivial. What is proper title case depends on the context. Sometimes there is not sufficient context to disambiguate between possibilities.

For example, consider `THE GAP OF THE OECD`. When title-cased, one might think this should become `The Gap of the OECD`—and that could be correct. However, "GAP" could also stand for _general accounting principles_, in which case the correct form would be `The GAP of the OECD`. Ultimately, it's unclear whether 'GAP' in the input is referring to a space between two things or to general accounting principles. (Perhaps the fact that it's followed by 'OECD', an organization which deals with economics and finance, pushes us slightly closer to reading it as general accounting principles.)

```
Original:  APPLE USA, INC. v. Jones, 644 F. Supp. 593 (E.D. N.Y., 1986)
Corrected: Apple USA, Inc. v. Jones, 644 F. Supp. 593 (E.D. N.Y., 1986)
```

#### Searches
Strafe can estimate _relative codeability_ using _weighted regex searches_. It does this by running a set of searches on the text of each case in a caselist.

If the user is using this functionality, first they write some searches and assign weights to them. It's actually not difficult to come up with decent searches.

Next, the user imports a caselist of cases that they want to run the searches on.

Then, the user opens all of the cases. As each case is opened, Strafe runs the searches on the case's text. Strafe gets each search's hit-count, multiplies it by that search's weight, then finds the sum. The result is a score for that case.

This score can then be compared against other cases' scores. (Of course, this requires that the same searches and weights are used.) This gives relative codeability. If the searches and weights are not terrible, the result is an estimate of relative codeability: **Cases with a higher score will be more likely to be codeable.**
