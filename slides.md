# Improving Research through Advanced REDCap Interfaces

### Scott Burns

`scott.s.burns@vanderbilt.edu`

This talk can be found at:

- Source: `http://github.com/sburns/advanced-redcap-interfaces`)
- Current Version: `http://bit.ly/advanced-redcap`

## Presenter Notes

---

# REDCap is

<br />

## A data storage service

<br />

## Secure for the storage of PHI

<br />

## An online spreadsheet

## Presenter Notes

---

# REDCap is not

<br />

## ...a real database:

- Not relational, can only track one "entity" per project
- Each Project is only a single table

## ...a spreadsheet because

- Client (web-app) & Server (PHP/MySQL) architecture
- Only requires a browser
- Advanced user access controls
- Advanced web features (why we're here today)

## Presenter Notes

I actually believe the REDCap team made a great decision to not incorporate relational semantics to REDCap. They would have lost a lot of potential users.

IMO, REDCap should be judged not as a less capable database but as a super spreadsheet that insulates users from many of the pitfalls of file-based databases.

---

# Advanced REDCap Features

<br />

## Application Programming Interface

External software can push and pull data over the internet

## Data Entry Triggers

Saving any data causes HTTP POST to URL (of your choosing)


## Presenter Notes

**These two features make REDCap the foundation for advanced data management workflows**

Investing in these workflows will improve your work. The techincal details of how these two features work are less important than the workflows they enable, which I'll talk about towards the end.

---

# Advanded data manag...what?

<br />

**Ideally, machines peform all trivial data analysis because they:**

*   Perform reproducible work.
*   Are extremely cheap.
*   Operate deterministically.

## Presenter Notes

As opposed to humans who:

*   Cannot be relied upon to reproduce their own work.
*   Are extremely expensive.
*   Some RAs might even be considered pseudorandom processes.

---

# Vocabulary

<br />

<table>
    <thead>
        <tr>
            <th class="left"><strong>REDCap</strong></th>
            <th><strong>SQL/Excel/etc</strong></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td class="left">Project</td>
            <td>Table</td>
        </tr>
        <tr>
            <td class="left">Data Dictionary</td>
            <td>Schema</td>
        </tr>
        <tr>
            <td class="left">Record</td>
            <td>Row</td>
        </tr>
        <tr>
            <td class="left">Field</td>
            <td>Column</td>
        </tr>
        <tr>
            <td class="left">Form or Instrument</td>
            <td>Set of columns</td>
        </tr>
        <tr>
            <td class="left">Unique Identifier</td>
            <td>Primary Key, Index, etc</td>
        </tr>

<!--         <tr>
            <td class="left"></td>
            <td></td>
        </tr>
 -->    </tbody>
</table>

---

# API


HTTP `POST` -> `https://redcap.vanderbilt.edu/api/`

`https://redcap.vanderbilt.edu/api/help/` describes payload parameters

## Note:

-   You need API access to the Project and a generated token
-   Available formats: `csv`, `xml`, `json`

## Presenter Notes

HTTP-based API for communicating with Projects

All API calls against REDCap are basically the same thing.

The API token binds your user rights to a specific project. DO NOT SHARE THIS TOKEN.

JSON (Javascript Object Notation) is the most interesting format from my perspective because it can be decoded to real objects.

---

# API

## Any language that has an HTTP library can communicate with REDCap.

`PyCap` is a python package I wrote and maintain to facilitate REDCap API usage within python applications/scripts.

`$ pip install PyCap`

`https://sburns.github.com/PyCap` for documentation.


## Presenter Notes

All languages worth using have an http library.

I'm going to talk about things implemented in python, but it's important to know that nothing about the REDCap API requires you to use python.

---

# API

## Most Important Methods

*   Export Data-Dictionary
*   Export, Import Records
*   Export/Import/Delete Files

---

# API: Exporting the Data-Dictionary


    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/ \
        -d token=XXX \
        -d content=metadata \
        -d format=json

<br />

    !python
    from redcap import Project
    project = Project('https://redcap.vanderbilt.edu/api/', TOKEN)
    md = project.export_metadata()


## Presenter Notes

While I'm talking about the API, I'm going to show curl commands towards the top and the corresponding python calls below.

---

# API: Exporting Data-Dictionary


-   Name
-   Label (Human Readable)
-   Type (Dropbown, Text, Textbox, etc)
-   Choices
-   Branching Logic (If any)
-   And others

Response is equivalent to downloading the data dictionary through the web application.

## Presenter Notes

A table about your table. Useful to determine whether fields exist before imports/etc.

---

# API: Exporting Records

Download the entire thing

    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/ \
        -d token=XXX \
        -d content=record \
        -d format=csv

<br />

    !python
    data = project.export_records()
    # data is a list of dictionary objects with one dictionary per record
    csv = project.export_records(format='csv') # or 'xml'
    df = project.export_records(format='df')  # Pandas DataFrame

---

# API: Exporting Data

Or request slices

    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/ \
        -d token=XXX \
        -d content=record \
        -d format=csv \
        -d records=1,2,3 \
        -d fields=age,sex,gender

<br />

    !python
    slices = project.export_records(records=['1', '2', '3'],
                                    fields=['age', 'sex', 'gender'])


## Presenter Notes

If you're working with a big database, you can speed up the API call by requesting only rows and columns that you'll need.

---

# API: Importing Data

Update existing records or add new ones

    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/ \
        -d token=XXX \
        -d content=record \
        -d format=csv \
        -d data="participant_id,new_data\n1,hooray"

<br />

    !python
    from my_module import modify_records
    data = project.export_records()
    modified = modify_records(data)
    project.import_records(modified)

## Presenter Notes

It's most likely best to download data, tweak it and import. By default does not overwrite with empty data.

---

# API: File Exporting

**Works on a per-record basis**

TODO

---

# API: File Importing

---

# API: File Deletion
